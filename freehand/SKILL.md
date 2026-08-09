---
name: freehand
description: Build something straight from a prompt - a short requirements interview, then implement it and run an adversarial build-and-review loop until an independent reviewer is satisfied. Use when the user wants to build a feature without writing a specification first, or mentions "/freehand".
argument-hint: "<what you want built>"
---

You build what the user asked for straight from their prompt: a short interview
to settle the requirements, then implementation, then an adversarial review loop
where an independent `freehand-reviewer` subagent attacks the work and you
incorporate its feedback until it is satisfied or the round cap is hit. The
reviewer tries to prove the implementation does not do what was asked; your job
is to leave it nothing to find.

This is the unspecced route: no specification bundle, no plan, no task list, and
nothing written to disk beyond the code itself. What replaces the specification
is the **transcript** - the user's prompt and the interview answers, verbatim,
handed to the reviewer as the contract. This skill is self-contained and does
not depend on any other skill.

## Inputs

`$ARGUMENTS` is the prompt, the whole of it treated as natural language. Do not
parse any part of it as an option. If it is empty, ask the user what they want
built before doing anything else.

The review round cap defaults to **3**, and changes only when the user asks in
words (e.g. "up to 5 rounds"). Never read a number in the prompt as the cap - it
almost always belongs to the request itself.

This skill commits as it goes (see **Committing**), so the working tree is clean
by the time the loop ends. Nothing reaches the remote - issues, the push, the
pull request - except through the delivery steps (9-11), each gated on the
user's approval.

## Procedure

1. **Explore before asking.** Read the codebase to establish the language,
   frameworks, conventions, project layout, test tooling, and the build, test
   and lint commands, from the project's own config (e.g. `package.json`
   scripts, `pyproject.toml`, build files) and its `CLAUDE.md`. Anything
   discoverable on disk is not a question.

2. **Grill: requirements only.** Interview the user about **what to build**, one
   question at a time, each with your recommended answer. Ask, wait for the
   reply, then work out what to ask next - never batch questions into one
   message, since an answer often removes the need for the next question.

   The default for an open question is to **decide it yourself**, not to ask.
   Ask only where the answer would materially change what gets built and you
   cannot infer it from the prompt or the codebase; otherwise choose the
   sensible option, proceed, and record the assumption for the final report. Two
   to five questions is typical, zero is a fine outcome for a clear prompt, and
   there is no quota to fill.

   Keep every question about requirements: scope and boundaries, what the user
   should be able to do, acceptance criteria, edge cases and error behaviour,
   what is explicitly out of scope. Never ask about the technical approach -
   language, frameworks, libraries, file layout, testing tools and project
   structure all come from step 1 or from your own judgement. Build what you are
   given: do not redirect the user to the `spec` skill, however large the
   request turns out to be, since choosing this route was their call. Write no
   files during this step.

3. **Assemble the transcript.** Before writing any code, assemble the contract
   you will hand the reviewer: the user's original prompt, then every interview
   exchange in order - the question you asked, the answer you recommended, and
   the user's reply. All of it **verbatim**. The recommendation matters, because
   a reply of "A" or "yes" means nothing without the question and the option it
   accepted. Never summarise, paraphrase, tidy or reorder it, then or later. The
   transcript is the only independent statement of what was asked for, and the
   moment it becomes your account of that, the review stops being independent.

4. **Implement.** Follow **test-driven development** for every behaviour change:
   write the test first, confirm it fails for the right reason, then implement
   until it passes. Commit each cohesive unit as it reaches a green state (see
   **Committing**).

   Where a change has no observable behaviour to assert - documentation,
   formatting, a configuration value carrying no logic - skip the test rather
   than write a tautological one, and note the skip for the final report so the
   reviewer can challenge it. That is the only exception, and it is narrow: if
   you can describe the change as a behaviour, it needs a test first.

5. **Run the mandatory checks.** Run the full build, test suite and linter,
   fixing failures until all are green and committing any fixes the full suite
   surfaced. Never proceed to review on red checks.

6. **Spawn the reviewer.** Use the Task tool with
   `subagent_type: freehand-reviewer`; its model and effort come from its own
   frontmatter, so do not override them here. Give it:
   - The **transcript** from step 3, in full and verbatim, every round. It has
     no memory between rounds and nothing on disk to read it from, so re-sending
     it is what makes the review possible at all.
   - Every assumption you decided rather than asked, and every test you skipped
     under step 4, each stated plainly so it can be attacked.
   - A summary of what you did this round, and the files or diff to inspect.
     This is context, not the contract - the transcript is the contract.
   - From round two onward, the feedback it gave last time, so it can confirm
     each point was genuinely addressed.

   Spawn it fresh each round; reviewing your own work inline defeats the
   mechanism. Present the work plainly - do not pre-defend it or steer it toward
   a pass.

7. **Read the verdict line.** `VERDICT: SATISFIED` ends the loop (go to step 9);
   `VERDICT: NEEDS_WORK` continues to step 8.

8. **Incorporate the feedback** in priority order, commit the round's fixes once
   the checks are green again, then return to step 5 - unless you have hit the
   round cap, in which case go to step 12.

9. **Collect unresolved issues.** This step and the two after it run only on
   `VERDICT: SATISFIED`; a capped or impasse finish goes straight to the
   report. Gather whatever the build left open - reviewer suggestions not acted
   on, items deferred as out of scope, TODOs the work introduced, limitations
   noticed along the way. If there are any, list them and ask the user which to
   create as issues on the project's GitHub repository, then create the chosen
   ones with `gh`. If nothing is open, say so and move on.

10. **Ask to deliver.** Ask the user for approval to push the branch and open a
    draft pull request. If the branch name carries a `worktree-` prefix,
    include a proposed replacement (a short name for what was built) in the ask
    and rename it with `git branch -m` before pushing. Without approval, the
    work stays local and you go straight to the report.

11. **See CI through.** Once pushed, watch the pull request's checks
    (`gh pr checks --watch` or the repository's equivalent) until they finish.
    On failure, diagnose, fix, commit and push again - step 10's approval
    covers follow-up pushes to the same branch - and watch the new run. Stop
    only on green checks or an impasse to bring back to the user.

12. **Report**: the rounds run and the final verdict, what changed and the
    commits made, every assumption you decided rather than asked, every test
    you skipped and why, any issues created, and the pull request and CI
    outcome where delivery was approved. Finish with a clean working tree
    either way. The work is _done_ only when all three hold: the mandatory
    checks pass, the implementation does everything the transcript asked for,
    and the reviewer returned `VERDICT: SATISFIED`. If you stopped on the cap
    instead, say so and list the reviewer's outstanding required changes.

## Committing

Commit continuously so the history reads as a sequence of self-contained steps,
not one large drop at the end.

- **Commit atomic units.** Each commit is one cohesive change that stands on its
  own. Small work is often a single commit; keep independent changes separate.
- **Only commit green.** Commit a unit once it builds and its tests pass, never
  a known-broken state. Under test-driven development a single commit carries
  the new tests and the implementation that satisfies them.
- **Commit review fixes too.** Each round commits the fixes it makes once the
  checks are green again (e.g. `Address review feedback: <summary>`).
- **Message style.** Imperative mood, concise subject (around 50 characters),
  body wrapped at 80 columns. No attribution footers.
- **Finish clean.** `git status` must be clean before you report - commit any
  stray change first.

## Visibility and guardrails

- Announce each phase as you enter it ("Exploring the codebase", "Implementing",
  "Round 2 of 3 - sending to reviewer") and report each verdict in one line.
- The cap is a hard limit. The loop ends as satisfied only when the reviewer
  itself returns `VERDICT: SATISFIED` - never declare success on its behalf.
- Delivery is opt-in: create only the issues the user selects, and never
  rename the branch, push or open the pull request without step 10's approval.
- If two consecutive rounds make no real progress on the same findings, stop
  early, report the impasse, and ask the user how to proceed.
