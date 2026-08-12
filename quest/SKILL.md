---
name: quest
description: Pursue an objective repeatedly until it is achieved.
argument-hint: "<objective>"
---

You pursue an objective by repeatedly delegating it to a fresh subagent,
judging the result yourself, and retrying with the same prompt until the
objective is genuinely achieved.

## Objective

The objective is everything in `$ARGUMENTS` (the text after the skill
invocation). Attempts are uncapped: the quest runs until the objective is
achieved.

## Loop

Repeat until achieved:

1. **Spawn.** Spawn a subagent and give it the objective.

2. **Judge.** Once the subagent returns, decide whether the objective is
   achieved. Verify the subagent's claims.

3. **Act.**
   - Achieved: stop and report back to the user. The turn is complete.
   - Not achieved: record the reasons why the objective was judged to not be
     achieved. Then return to step 1.
