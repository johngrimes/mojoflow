---
name: quest
description: Pursue an objective in a loop - delegate it to a fresh pi subprocess, judge the result against the objective yourself, and respawn with the same prompt until the objective is genuinely achieved. Use when the user gives an objective and wants it worked until done, or says "quest" or "keep trying until it works".
argument-hint: "<objective> [max-attempts]"
---

You pursue an objective by repeatedly delegating it to a fresh `pi` subprocess,
judging the result yourself, and retrying with the same prompt until the
objective is genuinely achieved. This is a single agent run, so commit your
changes as you go where the work calls for it.

## Objective

The objective is everything in `$ARGUMENTS` (the text after the skill
invocation). If it ends in an integer, that integer is the maximum number of
attempts and the rest is the objective. Otherwise attempts are uncapped: the
quest runs until the objective is achieved or the stall guard fires.

State the objective and the attempt cap in one line before starting.

## Loop

Repeat until achieved:

1. **Spawn.** From the current working directory, run:

   ```bash
   pi -p --no-session --no-skills "Task: <objective>"
   ```

   The objective text is always verbatim. From attempt two on, append a short
   `Previous attempts` section to the prompt: what each prior attempt did and
   why your verification judged it unsuccessful, so the fresh process can do
   better. The subprocess is a fresh pi with its own context, working in the
   same directory with the same tools. `--no-skills` stops it from re-entering
   this skill (or any other) and recursing. Quote the prompt so special
   characters survive; for very long objectives write the prompt to a temp file
   and pass it with `"$(cat /tmp/file)"`.

2. **Judge.** Decide whether the objective is achieved. Never trust the
   subprocess's claim - verify the actual state: run its tests, inspect the
   files it says it changed, exercise the result. The subprocess shares your
   working directory, so check the evidence directly.

3. **Act.**
   - Achieved: stop and go to Reporting.
   - Not achieved: record one or two lines on why, then go again - unless the
     attempt cap is reached, or the attempt made no progress over the previous
     one (identical failure, empty output, or a crash), in which case stop and
     ask the user how to proceed.

Announce each attempt as it starts, and give a one-line verdict after each
judgment.

## Reporting

When the loop ends, report:

- The objective and how many attempts it took.
- Per-attempt outcome: what was tried and why it was judged unsuccessful.
- The verification evidence for the final judgment: commands run, tests
  passing, files inspected.
- What changed in the working directory.

If the loop stopped without achieving the objective, say so plainly and list
what remains - never claim success.
