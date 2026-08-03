---
name: build
description: Implement a feature's specification one phase per invocation, then - once the final phase lands - run an adversarial build-and-review loop until an independent reviewer is satisfied. Use when the user wants to build or implement a spec produced by the spec skill, or mentions "/build".
argument-hint: "[spec-selector] [max-rounds]"
---

You implement a feature from the specification the `spec` skill produced, then
run an adversarial review loop: you build, an independent `build-reviewer`
subagent attacks the work against the spec, you incorporate its feedback - round
after round - until it is satisfied or the round cap is hit.

Where `tasks.md` has more than one phase, an invocation builds **one phase** -
the next one with unfinished tasks - and stops. The user runs the skill again to
take the next phase, which gives them a checkpoint between phases rather than
one large drop at the end.

The review loop runs **once**, on the invocation that completes the final phase,
with the whole feature in front of the reviewer. The spec's acceptance scenarios
and success criteria describe a finished feature, and a partly built one cannot
demonstrate them; a reviewer sent in early would be judging against a contract
the work is not yet meant to satisfy.

The review is deliberately adversarial. The reviewer tries to prove the
implementation does not meet the spec; your job is to leave it nothing to find.
This skill is self-contained and does not depend on any other skill.

## Inputs

Parse `$ARGUMENTS`:

- **Spec selector** (positional): a feature directory name or unique prefix
  (e.g. `003` or `003-user-auth`). If absent, default to the **latest** spec -
  the highest-numbered directory under `<repo-root>/.local/specs/`.
- **Max rounds**: default **3**. Only override when the user explicitly asks
  (e.g. "up to 5 rounds"); do not treat a positional integer as the cap, since
  the spec selector may itself be a number. It applies only to the invocation
  that reaches the review loop.
- **Phase**: default to the **next phase in `tasks.md` with unfinished tasks**.
  Only build a different phase when the user names one in words (e.g. "build
  phase 4"); as with the round cap, do not read a positional integer as a phase
  number.

Resolve the feature directory under `<repo-root>/.local/specs/`. If it has no
`tasks.md`, stop and tell the user to run `spec` first. State the resolved
feature directory, the phase you are about to build, how many phases remain
after it, and - where this is the last one - the round cap, back in one line
before starting.

This skill commits as it goes. The phase - or each cohesive group of tasks
within it - lands as its own atomic commit once its tests pass, and every review
round commits the fixes it makes, so every invocation ends with a clean working
tree, whether it stops at a phase boundary or at the end of the review loop. It
does not create branches or push to any remote; that stays with the user.

## Procedure

1. **Load the spec bundle.** Read `tasks.md` (required) and `plan.md`
   (required); read `spec.md`, `data-model.md`, `contracts/`, `research.md`,
   `quickstart.md`, and any `wireframes/` where present. Determine the project's
   build, test, and lint commands from `plan.md` and the project's own config
   (e.g. `package.json` scripts, `pyproject.toml`, build files).

2. **Resolve the phase.** Take the first phase in `tasks.md` order that still
   has `[ ]` tasks, unless the user named one. An unfinished task in an earlier
   phase means that earlier phase is the one to build - do not skip ahead to a
   later one. If every task is already `[X]` there is nothing left to build, so
   go straight to step 4: a feature whose last phase landed in an invocation
   that was interrupted still owes the user its review.

3. **Implement that phase, and only that phase.** Work in `tasks.md` order
   within it, respecting dependencies and `[P]` parallel markers, tasks on the
   same file run sequentially. Follow **test-driven development**: for each
   behaviour write the test first, confirm it fails for the right reason, then
   implement until it passes. Mark each task `[X]` in `tasks.md` as it genuinely
   completes, and commit as the work reaches a green state (see **Committing**
   below). Do not start a later phase's tasks, however trivial they look or
   however unfinished the phase feels without them; if one turns out to be a
   genuine prerequisite, note it in the report rather than absorbing it.

4. **Run the mandatory checks.** Run the full build, test suite, and linter.
   Fix failures until all are green. An invocation never ends on red checks and
   the review never starts on them. Commit any fixes the full suite surfaced
   before moving on.

5. **Stop here if any phase remains.** With work still outstanding in
   `tasks.md`, report the phase built, what changed, the commits made, and the
   phases still to come, then stop without spawning the reviewer. Leave a clean
   working tree. Only when every task in `tasks.md` is `[X]` do you continue to
   step 6.

6. **Spawn the reviewer.** Use the Agent tool with
   `subagent_type: build-reviewer`. Its model and effort are set in its own
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

9. **Report**: rounds run, final verdict, what changed, and the commits made.
   Whether the loop ends satisfied or on the cap, finish with a clean working
   tree - everything committed. The work is _done_ only when all three hold: the
   mandatory checks pass, every `tasks.md` item is genuinely complete, and the
   reviewer returned `VERDICT: SATISFIED`. If you stopped on the cap instead, say
   so and list the reviewer's outstanding required changes.

## Committing

Commit continuously so the history reads as a sequence of self-contained steps,
not one large drop at the end.

- **Commit atomic units.** Each commit is one coherent change that stands on its
  own - the phase itself where it is small, otherwise a cohesive group of its
  tasks that only make sense together. Keep independent changes in separate
  commits.
- **Only commit green.** Commit a unit when it builds and its tests pass; never
  commit a known-broken state. Under test-driven development this means a single
  commit carries the new tests, the implementation that satisfies them, and the
  matching `[X]` updates in `tasks.md`.
- **Commit review fixes too.** Each review round commits the changes it makes
  once the checks are green again (e.g. `Address review feedback: <summary>`).
- **Message style.** Imperative mood, concise subject (around 50 characters),
  body wrapped at 80 columns. No attribution footers.
- **Finish clean.** When the loop ends - satisfied or capped - nothing is left
  uncommitted; `git status` must be clean. Commit any stray change before
  reporting.

## Visibility and guardrails

- Announce the phase and how many remain as you start, and - in the invocation
  that reaches the review - announce each round before sending it ("Round 2 of
  3 - sending to reviewer") and report each verdict in one line.
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
