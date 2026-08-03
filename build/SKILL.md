---
name: build
description: Implement one phase of a feature's specification, then run an adversarial build-and-review loop until an independent reviewer is satisfied. Use when the user wants to build or implement a spec produced by the spec skill, or mentions "/build".
argument-hint: "[spec-selector] [max-rounds]"
---

You implement a feature from the specification the `spec` skill produced, then
run an adversarial review loop: you build, an independent `build-reviewer`
subagent attacks the work against the spec, you incorporate its feedback - round
after round - until it is satisfied or the round cap is hit.

Where `tasks.md` has more than one phase, an invocation builds **one phase** -
the next one with unfinished tasks - reviews it, and stops. The user runs the
skill again to take the next phase. This keeps each review round scoped to an
increment the reviewer can actually attack in depth, and gives the user a
checkpoint between phases rather than one large drop at the end.

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
  the spec selector may itself be a number.
- **Phase**: default to the **next phase in `tasks.md` with unfinished tasks**.
  Only build a different phase when the user names one in words (e.g. "build
  phase 4"); as with the round cap, do not read a positional integer as a phase
  number.

Resolve the feature directory under `<repo-root>/.local/specs/`. If it has no
`tasks.md`, stop and tell the user to run `spec` first. State the resolved
feature directory, the phase you are about to build, how many phases remain
after it, and the round cap back in one line before starting.

This skill commits as it goes. The phase - or each cohesive group of tasks
within it - lands as its own atomic commit once its tests pass, and every review
round commits the fixes it makes, so by the time the loop ends the working tree
is clean. It does not create branches or push to any remote; that stays with the
user.

## Procedure

1. **Load the spec bundle.** Read `tasks.md` (required) and `plan.md`
   (required); read `spec.md`, `data-model.md`, `contracts/`, `research.md`,
   `quickstart.md`, and any `wireframes/` where present. Determine the project's
   build, test, and lint commands from `plan.md` and the project's own config
   (e.g. `package.json` scripts, `pyproject.toml`, build files).

2. **Resolve the phase.** Take the first phase in `tasks.md` order that still
   has `[ ]` tasks, unless the user named one. An unfinished task in an earlier
   phase means that earlier phase is the one to build - do not skip ahead to a
   later one. If every task is already `[X]`, say the feature is complete and
   stop.

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
   Fix failures until all are green. Do not proceed to review with red checks.
   Commit any fixes the full suite surfaced before moving on.

5. **Spawn the reviewer.** Use the Agent tool with
   `subagent_type: build-reviewer`. Its model and effort are set in its own
   frontmatter, so do not override them here. Give it the feature directory
   path, the phase under review and its task IDs, a summary of what you did this
   round, the files or diff to inspect, and - from round two onward - the
   feedback it gave last time so it can confirm each point was addressed. State
   plainly that later phases are unbuilt and out of scope for this round. Spawn
   it fresh each round; reviewing your own work inline defeats the mechanism.
   Present the work plainly - do not pre-defend it or steer it toward a pass.

6. **Read the verdict line.** `VERDICT: SATISFIED` ends the loop (go to step 8);
   `VERDICT: NEEDS_WORK` continues to step 7.

7. **Incorporate the feedback** in priority order, committing the round's fixes
   once the checks are green again, then return to step 4 - unless you have hit
   the round cap, in which case stop.

8. **Report**: the phase built, rounds run, final verdict, what changed, and the
   commits made. Name the phases still outstanding and say that running the
   skill again picks up the next one; if this was the last phase, say the
   feature is complete instead. Whether the loop ends satisfied or on the cap,
   finish with a clean working tree - everything committed. The phase is _done_
   only when all three hold: the mandatory checks pass, every task in that phase
   is genuinely complete, and the reviewer returned `VERDICT: SATISFIED`. If you
   stopped on the cap instead, say so and list the reviewer's outstanding
   required changes.

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

- Announce each round before sending it ("Round 2 of 3 - sending to reviewer")
  and report each verdict in one line.
- One phase per invocation is as hard a limit as the round cap. Stop once the
  phase is reviewed - do not roll on into the next one because the checks are
  green, the phase was small, or the remaining work looks obvious.
- The cap is a hard limit. The loop ends as satisfied only when the reviewer
  itself returns `VERDICT: SATISFIED` - never declare success on its behalf.
- If two consecutive rounds make no real progress on the same findings, stop
  early, report the impasse, and ask the user how to proceed.
