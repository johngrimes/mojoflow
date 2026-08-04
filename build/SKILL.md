---
name: build
description: Implement a feature's specification one phase per invocation, then - once the final phase lands - run an adversarial build-and-review loop until an independent reviewer is satisfied. Use when the user wants to build or implement a spec produced by the spec skill, or mentions "/build".
argument-hint: "[spec-selector] [max-rounds]"
---

You implement a feature from the specification the `spec` skill produced, then
run an adversarial review loop: you build, an independent `build-reviewer`
subagent attacks the work against the spec, you incorporate its feedback - round
after round - until it is satisfied or the round cap is hit. The reviewer tries
to prove the implementation does not meet the spec; your job is to leave it
nothing to find.

An invocation builds **one phase** of `tasks.md`, the next one with unfinished
tasks, and stops - a checkpoint between phases rather than one large drop at the
end. The review loop runs **once**, on the invocation that completes the final
phase, with the whole feature in front of the reviewer: the spec's acceptance
scenarios and success criteria describe a finished feature, and a partly built
one cannot demonstrate them.

This skill is self-contained and does not depend on any other skill.

## Inputs

Parse `$ARGUMENTS`:

- **Spec selector** (positional): a feature directory name or unique prefix
  (e.g. `003` or `003-user-auth`). Defaults to the highest-numbered directory
  under `<repo-root>/.local/specs/`.
- **Max rounds**: default **3**, and changes only when the user asks in words
  (e.g. "up to 5 rounds"). It applies only to the invocation that reaches the
  review loop.
- **Phase**: defaults to the next phase in `tasks.md` with unfinished tasks;
  build a different one only when the user names it in words (e.g. "build phase
  4").

Never read a positional integer as the round cap or the phase - the spec
selector may itself be a number.

Resolve the feature directory under `<repo-root>/.local/specs/`. If it has no
`tasks.md`, stop and tell the user to run `spec` first. Before starting, state
in one line: the resolved directory, the phase you are about to build, how many
phases remain after it, and - where this is the last one - the round cap.

This skill commits as it goes (see **Committing**), so every invocation ends
with a clean working tree, whether it stops at a phase boundary or at the end of
the review loop. It does not create branches or push to any remote; that stays
with the user.

## Procedure

1. **Load the spec bundle.** Read `tasks.md` and `plan.md` (both required), plus
   `spec.md`, `data-model.md`, `contracts/`, `research.md`, `quickstart.md` and
   `wireframes/` where present. Determine the build, test and lint commands from
   `plan.md` and the project's own config (e.g. `package.json` scripts,
   `pyproject.toml`, build files).

2. **Resolve the phase.** The first phase in `tasks.md` order that still has
   `[ ]` tasks, unless the user named one - an unfinished task in an earlier
   phase means that earlier phase is the one to build, so do not skip ahead. If
   every task is already `[X]`, go straight to step 4: a feature whose last
   phase landed in an interrupted invocation still owes the user its review.

3. **Implement that phase, and only that phase.** Work in `tasks.md` order
   within it, respecting dependencies and `[P]` parallel markers; tasks on the
   same file run sequentially. Follow **test-driven development**: for each
   behaviour write the test first, confirm it fails for the right reason, then
   implement until it passes. Mark each task `[X]` as it genuinely completes,
   and commit as the work reaches a green state. Do not start a later phase's
   tasks, however trivial they look or however unfinished the phase feels
   without them; if one turns out to be a genuine prerequisite, note it in the
   report rather than absorbing it.

4. **Run the mandatory checks.** Run the full build, test suite and linter,
   fixing failures until all are green and committing any fixes the full suite
   surfaced. An invocation never ends on red checks and the review never starts
   on them.

5. **Stop here if any phase remains.** Report the phase built, what changed, the
   commits made and the phases still to come, then stop without spawning the
   reviewer, leaving a clean working tree. Continue to step 6 only when every
   task in `tasks.md` is `[X]`.

6. **Spawn the reviewer.** Use the Agent tool with
   `subagent_type: build-reviewer`; its model and effort come from its own
   frontmatter, so do not override them here. The whole feature is under review,
   not just the phase you just finished: give it the feature directory path, a
   summary of the work across every phase, the full diff for the feature (which
   may span several invocations and many commits), and - from round two onward -
   the feedback it gave last time so it can confirm each point was addressed.
   Spawn it fresh each round; reviewing your own work inline defeats the
   mechanism. Present the work plainly - do not pre-defend it or steer it toward
   a pass.

7. **Read the verdict line.** `VERDICT: SATISFIED` ends the loop (go to step 9);
   `VERDICT: NEEDS_WORK` continues to step 8.

8. **Incorporate the feedback** in priority order, re-run the mandatory checks
   until green and commit the round's fixes, then return to step 6 - unless you
   have hit the round cap, in which case stop.

9. **Report**: rounds run, final verdict, what changed, and the commits made,
   finishing with a clean working tree either way. The work is _done_ only when
   all three hold: the mandatory checks pass, every `tasks.md` item is genuinely
   complete, and the reviewer returned `VERDICT: SATISFIED`. If you stopped on
   the cap instead, say so and list the reviewer's outstanding required changes.

## Committing

Commit continuously so the history reads as a sequence of self-contained steps,
not one large drop at the end.

- **Commit atomic units.** Each commit is one coherent change that stands on its
  own - the phase itself where it is small, otherwise a cohesive group of its
  tasks that only make sense together. Keep independent changes separate.
- **Only commit green.** Commit a unit once it builds and its tests pass, never
  a known-broken state. Under test-driven development a single commit carries
  the new tests, the implementation that satisfies them, and the matching `[X]`
  updates in `tasks.md`.
- **Commit review fixes too.** Each round commits the fixes it makes once the
  checks are green again (e.g. `Address review feedback: <summary>`).
- **Message style.** Imperative mood, concise subject (around 50 characters),
  body wrapped at 80 columns. No attribution footers.
- **Finish clean.** `git status` must be clean before you report - commit any
  stray change first.

## Visibility and guardrails

- Announce the phase and how many remain as you start, and - in the invocation
  that reaches the review - each round before sending it ("Round 2 of 3 -
  sending to reviewer"), then report each verdict in one line.
- One phase per invocation is as hard a limit as the round cap. Stop once the
  phase is green and committed - do not roll on into the next one because the
  phase was small or the remaining work looks obvious.
- The review is neither early nor optional. Do not spawn the reviewer while any
  `tasks.md` item is unfinished, and do not end the invocation that completes
  the last phase without running the loop.
- The cap is a hard limit. The loop ends as satisfied only when the reviewer
  itself returns `VERDICT: SATISFIED` - never declare success on its behalf.
- If two consecutive rounds make no real progress on the same findings, stop
  early, report the impasse, and ask the user how to proceed.
