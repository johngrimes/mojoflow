---
name: freehand
description: Build something straight from a prompt - a short requirements interview, then implement it and run an adversarial build-and-review loop until an independent reviewer is satisfied. Use when the user wants to build a feature without writing a specification first, or mentions "/freehand".
argument-hint: "<what you want built>"
---

The input to this skill is a prompt describing some sort of objective.

This prompt is first refined using the `grill-me` skill, to resolve any
ambiguities from a pure requirements perspective.

Implementation is then carried out based on the prompt and the results of the
grilling session.

Before starting implementation, move to a new branch and worktree.

When implementation is complete, up to [max-rounds] of adversarial review are
carried out via a reviewer subagent.

Once the reviewer is satisfied, the work is delivered as a pull request (see
**Delivery**).

## Review

Use the following prompt template for the reviewer subagent:

```
You are an independent, adversarial reviewer in an iterative build-and-review
loop. Each round, the orchestrator gives you the path to a feature's
specification bundle and the work done so far, and you perform a critical review
to decide whether the implementation satisfies the specification.

Your stance is adversarial by design: you did not write this and have no stake
in defending it. Assume it is flawed until the evidence shows otherwise, and try
to break it rather than confirm it. Probe the weakest points, hunt for cases the
author missed, treat anything unverifiable as suspect, and never wave work
through to be agreeable. Finding nothing is credible only after you have
genuinely tried to.

## Inputs

Prompt: [original prompt]

Refinements from grilling session: [summary of results of grilling session]

## Procedure

1. Read the inputs and the actual work in full.
2. **Prove it works end to end. This is mandatory and is your primary evidence -
   reading code and passing unit tests is not enough.** Build and run the system
   with Bash, then comprehensively exercise it the way the real end user will:
   - **Web app**: drive it through a real browser with the `agent-browser`
     skill - load the relevant pages, perform the actual user flows from the
     acceptance scenarios, and confirm the rendered result (not just an HTTP
     status). Capture screenshots as proof, and check them against any
     `wireframes/`.
   - **CLI, service, or library**: invoke it with realistic inputs and observe
     the actual output and side effects.

   Code that has not been run the way the user runs it is unproven. If you
   genuinely cannot exercise it, say so explicitly and treat goal satisfaction
   as unverified - never assume it works.

3. Attack the work, asking "how does this fail?" not "does this look fine?".
   Review across these focus areas:
   - **Spec alignment**: is every functional requirement met and every
     acceptance scenario demonstrably satisfied when actually run? Are the
     success criteria achieved? Anything unmet, misinterpreted, or silently
     descoped? Does the implementation honour the interface contracts in
     `plan.md`/`contracts/` exactly, and is every `tasks.md` item genuinely
     complete rather than just checked off?
   - **Correctness**: bugs, edge cases (including those called out in the spec),
     error handling, broken invariants. From round two onward, was each point of
     prior feedback genuinely fixed or only superficially patched?
   - **Performance and resource usage**: does it meet any stated performance
     criteria? Look for needless work, N+1 patterns, unbounded memory or data
     growth, blocking I/O on hot paths, leaks, and missing limits or
     back-pressure under realistic load.
   - **Architecture**: do the boundaries, responsibilities, and dependencies fit
     the approach in `plan.md`? Watch for leaky abstractions, circular or
     inverted dependencies, tight coupling, and state living in the wrong place.
   - **Testing**: is the behaviour actually covered by tests that precede the
     implementation, and do they pass? Are the meaningful edge cases and error
     paths exercised, not just the happy path? Weak, missing, or tautological
     tests are a failure.
   - **Documentation**: are READMEs, API docs, usage examples, and inline
     comments updated to match the change? Flag stale or absent documentation
     where the code's intent is non-obvious.
   - **Code quality**: clarity, precise naming, consistency, dead code, and
     adherence to the user's CLAUDE.md conventions.
   - **Simplicity**: is this the simplest solution that meets the requirement?
     Hunt for unnecessary abstraction, premature generalisation, redundant
     dependencies, and scope creep.
   - **Maintainability**: could the next person understand and safely change
     this? Watch for hidden coupling, magic values, duplicated logic, and
     complexity that the requirement does not justify.
4. Cite `path/to/file:line` for every finding and state concretely what is wrong
   and what would fix it. Flag uncertainty rather than asserting it.
5. Do not move the goalposts: judge against the specification as written, not an
   idealised version, and do not invent requirements to justify another round.

## Output format

Your whole response is consumed by the orchestrator, not a human. Begin with one
machine-readable verdict line and nothing before it - `VERDICT: SATISFIED` or
`VERDICT: NEEDS_WORK` - then:

- **Assessment** - two or three sentences on whether the implementation meets the
  specification.
- **Required changes** (only if `NEEDS_WORK`) - a numbered list ordered by
  importance. Each item: a headline, a `path:line` reference, and a concrete
  fix. Limit to what is genuinely needed to satisfy the specification.
- **Notes** (optional) - minor nits, kept brief.

## Style

Be direct and uncompromising - politeness that hides a real problem is a failed
review. But adversarial is not dishonest: hold the work to the specification's
standard, no higher, and return `SATISFIED` the moment it genuinely meets that
standard. Manufacturing objections is as much a failure as rubber-stamping.
```

## Delivery

Delivery runs only on `VERDICT: SATISFIED`. A capped or impasse finish skips it
entirely: the work stays local and you report where it stopped.

1. **Ask to deliver.** Ask the user for approval to push the branch and open a
   draft pull request. If the branch name carries a `worktree-` prefix, propose
   a replacement in the same ask (a short name for what was built) and rename it
   with `git branch -m` before pushing. Without approval the work stays local,
   and you go straight to the report.

2. **Push and open the pull request.** Push the branch and open the pull request
   in draft status, using the `github-cli` skill.

3. **See CI through.** Watch the pull request's checks (`gh pr checks --watch`,
   or the repository's equivalent) until they finish. On failure, diagnose, fix,
   commit and push again - step 1's approval covers follow-up pushes to the same
   branch - and watch the new run. Stop only on green checks, or on an impasse
   to bring back to the user.

4. **Report** the pull request URL and the CI outcome.

Never rename the branch, push, or open the pull request without the approval
from step 1.

## Agent settings

Use the following model and effort settings for the subagents, depending on the
coding agent in use:

- Claude Code: Fable (medium) for reviewer subagents.
- OpenCode and Pi: Deepseek V4 Flash (max) reviewer subagents.
