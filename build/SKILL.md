---
name: build
description: Implement a feature's specification in full, then run an adversarial build-and-review loop until an independent reviewer is satisfied. Use when the user wants to build or implement a spec produced by the spec skill, or mentions "/build".
argument-hint: "[spec-selector] [max-rounds]"
---

You implement a feature from the specification the `spec` skill produced, then
run an adversarial review loop: you build, an independent `build-reviewer`
subagent attacks the work against the spec, you incorporate its feedback - round
after round - until it is satisfied or the round cap is hit. The reviewer tries
to prove the implementation does not meet the spec; your job is to leave it
nothing to find.

You build every phase of `tasks.md`, and the review loop starts only once every
task is complete, with the whole feature in front of the reviewer: the spec's
acceptance scenarios and success criteria describe a finished feature, and a
partly built one cannot demonstrate them.

This skill is self-contained and does not depend on any other skill.

## Inputs

Parse `$ARGUMENTS`:

- **Spec selector** (positional): a feature directory name or unique prefix
  (e.g. `003` or `003-user-auth`). Defaults to the highest-numbered directory
  directly under `<repo-root>/.local/specs/`, which holds the outstanding specs
  only. `archive/` is not a feature directory and is never resolved: if the
  selector matches nothing outstanding but does match an archived spec, say it
  has already been built and ask the user whether to move it back out of
  `archive/` before building again.
- **Max rounds**: default **3**, and changes only when the user asks in words
  (e.g. "up to 5 rounds").

Never read a positional integer as the round cap - the spec selector may itself
be a number.

Resolve the feature directory under `<repo-root>/.local/specs/`. If it has no
`tasks.md`, stop and tell the user to run `spec` first. Before starting, state
in one line: the resolved directory, how many phases it holds and the round cap.

This skill commits as it goes (see **Committing**), so the loop ends with a
clean working tree, satisfied or capped. Nothing reaches the remote - issues,
the push, the pull request - except through the delivery steps (8-10), each
gated on the user's approval.

## Procedure

1. **Load the spec bundle.** Read `tasks.md` and `plan.md` (both required), plus
   `spec.md`, `data-model.md`, `contracts/`, `research.md`, `quickstart.md` and
   `wireframes/` where present. Determine the build, test and lint commands from
   `plan.md` and the project's own config (e.g. `package.json` scripts,
   `pyproject.toml`, build files).

2. **Implement every phase** in `tasks.md` order, respecting dependencies and
   `[P]` parallel markers; tasks on the same file run sequentially. Follow
   **test-driven development**: for each behaviour write the test first, confirm
   it fails for the right reason, then implement until it passes. Mark each task
   `[X]` as it genuinely completes, report progress per phase, and commit as the
   work reaches a green state. If every task is already `[X]` - a build
   interrupted after its last task - there is nothing left to build, and the
   feature still owes the user its review.

3. **Run the mandatory checks.** Run the full build, test suite and linter,
   fixing failures until all are green and committing any fixes the full suite
   surfaced. The review never starts on red checks.

4. **Spawn the reviewer.** Use the Task tool with
   `subagent_type: build-reviewer`; its model and effort come from its own
   frontmatter, so do not override them here. Give it the feature directory
   path, a summary of the work across every phase, the full diff for the
   feature, and - from round two onward - the feedback it gave last time so it
   can confirm each point was addressed. Spawn it fresh each round; reviewing
   your own work inline defeats the mechanism. Present the work plainly - do not
   pre-defend it or steer it toward a pass.

5. **Read the verdict line.** `VERDICT: SATISFIED` ends the loop (go to step 7);
   `VERDICT: NEEDS_WORK` continues to step 6.

6. **Incorporate the feedback** in priority order, re-run the mandatory checks
   until green and commit the round's fixes, then return to step 4 - unless you
   have hit the round cap, in which case go to step 7.

7. **Archive the spec** - only on `VERDICT: SATISFIED`. Move the whole feature
   directory into `<repo-root>/.local/specs/archive/`, creating that directory
   if it does not exist and keeping the `NNN-short-name` name unchanged, so
   `.local/specs/` is left holding only work that is still outstanding. These
   files are outside version control, so move them with `mv`, not `git mv`. If
   you stopped on the cap or on an impasse, leave the directory where it is -
   the feature is still in progress.

8. **Collect unresolved issues.** This step and the two after it run only on
   `VERDICT: SATISFIED`; a capped or impasse finish goes straight to the
   report. Gather whatever the build left open - reviewer suggestions not acted
   on, items deferred as out of scope, TODOs the work introduced, limitations
   noticed along the way. If there are any, list them and ask the user which to
   create as issues on the project's GitHub repository, then create the chosen
   ones with `gh`. If nothing is open, say so and move on.

9. **Ask to deliver.** Ask the user for approval to push the branch and open a
   draft pull request. If the branch name carries a `worktree-` prefix, include
   a proposed replacement (e.g. the spec directory's `NNN-short-name`) in the
   ask and rename it with `git branch -m` before pushing. Without approval, the
   work stays local and you go straight to the report.

10. **See CI through.** Once pushed, watch the pull request's checks
    (`gh pr checks --watch` or the repository's equivalent) until they finish.
    On failure, diagnose, fix, commit and push again - step 9's approval covers
    follow-up pushes to the same branch - and watch the new run. Stop only on
    green checks or an impasse to bring back to the user.

11. **Report**: rounds run, final verdict, what changed, the commits made,
    where the spec directory now sits, any issues created, and the pull request
    and CI outcome where delivery was approved - finishing with a clean working
    tree either way. The work is _done_ only when all three hold: the mandatory
    checks pass, every `tasks.md` item is genuinely complete, and the reviewer
    returned `VERDICT: SATISFIED`. If you stopped on the cap instead, say so
    and list the reviewer's outstanding required changes.

## Committing

Commit continuously so the history reads as a sequence of self-contained steps,
not one large drop at the end.

- **Commit atomic units.** Each commit is one coherent change that stands on its
  own - a phase where it is small, otherwise a cohesive group of its tasks that
  only make sense together. Keep independent changes separate.
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

- Announce each phase as you start it, then each review round before sending it
  ("Round 2 of 3 - sending to reviewer"), and report each verdict in one line.
- The review is neither early nor optional. Do not spawn the reviewer while any
  `tasks.md` item is unfinished, and do not finish the build without running the
  loop.
- The cap is a hard limit. The loop ends as satisfied only when the reviewer
  itself returns `VERDICT: SATISFIED` - never declare success on its behalf.
- Delivery is opt-in: create only the issues the user selects, and never
  rename the branch, push or open the pull request without step 9's approval.
- If two consecutive rounds make no real progress on the same findings, stop
  early, report the impasse, and ask the user how to proceed.
