---
name: goal
description: Work towards a goal until its acceptance criteria are met, with a bounded self-review/repair loop. Use when the user says "/goal", "goal", "active goal", "continue until evidence says done", or "verify against a goal".
argument-hint: "<goal description>"
---

# Goal

When the user invokes this skill - by saying `/goal`, "goal", "active goal",
"continue until evidence says done", or "verify against a goal" - you
transition into goal-driven execution: define explicit acceptance criteria,
work towards them, self-review against evidence, repair until satisfied or
capped, and report the outcome.

This skill is self-contained. It does not call any external CLI or other skill.

## Inputs

`$ARGUMENTS` is the natural-language goal description. If empty, ask the user
what goal they want to achieve before doing anything else.

## Procedure

The flow is **clarify → contract → implement → review/repair → report**, run as
an explicit bounded loop. Every iteration after implementation includes a
self-review against the contract's evidence requirements, followed by repair if
criteria are not yet met.

### 1. Clarify the goal

If the goal description is vague, interview the user with one question at a time
until you have concrete, testable acceptance criteria. Each question should
include your recommended answer. Cover at least:

- What must be true for the goal to be considered done?
- What evidence will prove each criterion is met?
- What commands or checks can verify the evidence?
- What is out of scope (hard constraints)?
- How many self-review/repair rounds should we allow before stopping (default: 3)?

Do not write any files during this step. Stop when the acceptance criteria are
unambiguous and testable, not before.

### 2. Write the goal contract

Create a goal contract document at `.local/goal/contract.md` (relative to the
repository root). The contract must contain:

1. **Goal statement**: a one-sentence summary of what will be achieved.
2. **Acceptance criteria**: a numbered list of concrete, testable criteria.
   Each criterion must be binary - it either passes or fails with no ambiguity.
3. **Evidence plan**: for each criterion, what evidence will be collected
   (changed files, tests added, commands run, validation output, etc.).
4. **Verification commands**: the exact commands to run to verify the evidence,
   with expected exit codes.
5. **Stop rules**: hard constraints that will cause the loop to stop regardless
   of progress (e.g. "do not edit unrelated files", "stop if a product decision
   is needed").
6. **Round budget**: maximum self-review/repair rounds (default: 3).

### 3. Implement

Work towards the goal. Follow test-driven development: write tests first, verify
they fail, then implement until they pass. Keep changes minimal and focused on
the acceptance criteria. Do not widen scope beyond the contract.

After implementation:

- Report what was changed and what was left undone.
- Run the verification commands and record exit codes and output.
- Flag any surprises, new risks, or decisions that need the user's approval.

### 4. Self-review against the contract

After implementation, review your own work against the acceptance criteria:

1. Read every acceptance criterion from the contract.
2. For each criterion, check the actual evidence (changed files, test results,
   command output) against the evidence plan.
3. Mark each criterion as **PASS** (evidence confirms it is met) or **FAIL**
   (evidence is missing or contradicts it).
4. Record your findings in `.local/goal/review-N.md` (where N is the review
   round number, starting at 1).

If all criteria pass, proceed to step 6 (Report).

### 5. Repair

If any criteria failed the review:

1. Identify the smallest set of changes that will make the failing criteria pass.
2. Apply those changes.
3. Re-run the verification commands.
4. Increment the round counter and repeat from step 4.

Stop repairing when:

- All criteria pass (success).
- The round budget is exhausted without all criteria passing (give up and report).
- A stop rule is triggered (e.g. an unapproved product decision is needed).
- The user interrupts to provide guidance.

Do not repair for optional polish or criteria that were not in the contract.

### 6. Report

Write a final report at `.local/goal/report.md` summarising:

- The goal statement.
- Final verdict: **SATISFIED** (all criteria met) or **NOT SATISFIED** (some
  criteria unmet after max rounds).
- Per-criterion status with evidence references.
- Rounds run and what changed in each round.
- Verification command results.
- Residual risks or skipped checks.
- Any decisions that need the user's attention.

Tell the user the verdict, the contract path, and - if not satisfied - which
criteria remain unmet and their recommendation for next steps.

## Subagent integration

When work can be delegated to a subagent (e.g. using `subagent()` with a
`worker`), pass the acceptance criteria from the goal contract as a structured
`acceptance` parameter:

```typescript
subagent({
  agent: "worker",
  task: "Implement the goal: <goal description>...",
  acceptance: {
    criteria: [
      "Criterion 1 from the goal contract",
      "Criterion 2 from the goal contract"
    ],
    evidence: ["changed-files", "commands-run", "validation-output"],
    verify: [
      { id: "tests", command: "npm test" },
      { id: "lint", command: "npm run lint" }
    ],
    stopRules: [
      "Do not edit unrelated files",
      "Stop if an unapproved product decision is needed"
    ],
    maxFinalizationTurns: 3
  },
  async: true
})
```

This delegates the implement-review-repair loop to the subagent, which is the
closest pi equivalent to Claude Code's native goal loop. The parent should still
read the subagent's result, verify the evidence, and report the outcome.

## Visibility

Announce each phase as you enter it: "Clarifying the goal", "Writing the goal
contract", "Implementing", "Self-review round N", "Repairing", "Reporting".
Keep the user informed of progress and verdict.
