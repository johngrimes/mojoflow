# MojoFlow

A small pair of Claude Code skills for an agentic coding workflow, taking a
feature from a rough idea to a finished, reviewed implementation in two moves:

1. **`spec`** - interview yourself to a shared understanding, then write a
   complete, self-contained specification bundle.
2. **`build`** - implement that bundle, then run an adversarial build-and-review
   loop until an independent reviewer is satisfied.

The two skills hand off cleanly: `spec` produces a feature directory under the
project's `.local/specs/`, and `build` consumes it.

The overall shape - the spec → plan → tasks progression, the artifact set
(`spec.md`, `plan.md`, `tasks.md`, `research.md`, `data-model.md`, `contracts/`,
`quickstart.md`), the sequential `NNN-short-name` feature directories, and the
project constitution - draws on GitHub's
[spec-kit](https://github.com/github/spec-kit).

A third skill, **`constitution`**, is a companion to the flow rather than a step
in it: it drafts or amends a project's non-negotiable principles and writes them
into `CLAUDE.md`, where `spec` and `build` then honour them.

## What's here

```
spec/                       The /spec skill
  SKILL.md
  templates/                spec, plan, tasks, and checklist templates
  wireframing/              lo-fi HTML wireframe template and UI patterns
build/                      The /build skill
  SKILL.md
constitution/               The /constitution skill
  SKILL.md
  references/               constitution template
agents/
  build-reviewer.md         Independent adversarial reviewer used by /build
```

## The skills

### spec

Turns a feature idea into a complete specification. The flow is
**grill → spec → grill → plan → tasks**, run as a soft flow with no hard
approval gates:

1. **Grill** on requirements, scope, and UI - one question at a time, each with
   a recommended answer - until the requirements are unambiguous.
2. Create the next sequential feature directory `NNN-short-name` under
   `<repo-root>/.local/specs/`.
3. Write `spec.md` (what and why, not how) plus a requirements checklist.
4. Generate lo-fi HTML wireframes into `wireframes/`, if the feature has a UI.
5. **Grill** on the technical approach - stack, frameworks, storage, testing,
   structure, interface contracts.
6. Write `plan.md` with a constitution check, plus `research.md`,
   `data-model.md`, `contracts/`, and `quickstart.md` where relevant.
7. Write a dependency-ordered, test-first `tasks.md`.
8. Run a consistency check across the artifacts, fixing any issues it surfaces.

The skill is self-contained: it owns its templates and wireframing assets and
calls no external CLI or other skill.

#### Why grill instead of just reading a spec

Reading a large specification is an unreliable way to build shared
understanding. A long document invites skimming, lets ambiguity hide in prose,
and gives no signal about which parts were actually absorbed. The interview
replaces that with a focused exchange: one question at a time, each forcing a
concrete decision, until nothing material is left unresolved.

It follows that the resulting spec is written for the agent, not a human reader

- a precise, machine-consumable record of the decisions reached, and the input
  `build` works from. There is little value in committing these specs: they are
  an artifact of a particular agent session, not source. They live under
  `.local/`, which is ignored by Git globally, and stay out of version control
  by design.

The grilling idea comes from Matt Pocock's
[Grill Me skill](https://www.aihero.dev/skills-grill-me).

### build

Implements a feature from the `spec` bundle, then runs an adversarial review
loop:

1. Load the spec bundle and determine the project's build, test, and lint
   commands.
2. Implement every phase in `tasks.md` order, following test-driven development,
   marking each task done as it genuinely completes.
3. Run the full build, test suite, and linter until green.
4. Spawn a fresh `build-reviewer` subagent that attacks the work against the
   spec.
5. Read its verdict line: `SATISFIED` ends the loop; `NEEDS_WORK` continues.
6. Incorporate the feedback and loop, up to the round cap (default 3).
7. Report the rounds run, final verdict, and what changed.

Work is done only when the mandatory checks pass, every `tasks.md` item is
genuinely complete, and the reviewer returns `VERDICT: SATISFIED`.

### build-reviewer agent

An independent, read-only, deliberately adversarial reviewer. It reads the
specification bundle directly (never the orchestrator's summary), proves the
implementation works end to end by running it, attacks the weakest points, and
returns a machine-readable verdict plus an ordered list of required changes.
`build` spawns it fresh each round.

### constitution

Drafts or amends a project's constitution - a short set of non-negotiable
principles that govern how the project is specified, planned, built and
reviewed:

1. Examine the project (existing `CLAUDE.md`, `README.md`, build files, source
   layout) to build a picture before drafting.
2. Confirm scope with a tight interview - fresh draft or amendment, how many
   principles, any non-negotiables to capture verbatim.
3. Draft declarative, testable principles from the template, one concern each,
   with a short rationale where the reason is not obvious.
4. Write the result into a single `## Constitution` section of `CLAUDE.md`, so
   it loads into every Claude Code session automatically.

Because the constitution lives in `CLAUDE.md`, both `spec` (which runs a
constitution check while planning) and `build` pick it up without any extra
wiring.

## Installation (Claude Code)

These are standard Claude Code skills. Place the `spec`, `build` and
`constitution` directories under a skills directory Claude Code reads - typically
`~/.claude/skills/` for personal use or `.claude/skills/` within a project - and
place `agents/build-reviewer.md` under the matching `agents/` directory so
`build` can find its reviewer.

## Usage

```
/spec <feature description>
/build [spec-selector] [max-rounds]
```

`spec` writes its artifacts under `<repo-root>/.local/specs/NNN-short-name/`.
`build` defaults to the latest spec when no selector is given, and to a cap of
three review rounds.

## Using with OpenCode

[OpenCode](https://opencode.ai) reads the same `SKILL.md` format natively, so
both skills run there with one adaptation for the reviewer agent.

### Install the skills

OpenCode discovers skills by walking up from the working directory to the git
root, scanning (among others) `.claude/skills/<name>/SKILL.md` and
`~/.claude/skills/<name>/SKILL.md` - the same locations Claude Code uses. So if
the skills are already installed for Claude Code, OpenCode finds them with no
further work. Otherwise place them under one of OpenCode's own paths:

- Project: `.opencode/skills/spec/` and `.opencode/skills/build/`
- Global: `~/.config/opencode/skills/spec/` and `~/.config/opencode/skills/build/`

The `name` and `description` frontmatter both skills carry are the only required
fields; the Claude-specific `argument-hint` is ignored by OpenCode.

### Enable them

Skills are gated by pattern in `opencode.json`. Allow these two explicitly:

```json
{
  "permission": {
    "skill": {
      "spec": "allow",
      "build": "allow"
    }
  }
}
```

Values are `allow`, `deny`, or `ask`, and patterns support wildcards.

### Adapt the reviewer agent

The `build` loop spawns a `build-reviewer` subagent. The provided
`agents/build-reviewer.md` uses Claude Code's agent frontmatter, which OpenCode
does not read. Create an OpenCode subagent at `.opencode/agent/build-reviewer.md`
(or `~/.config/opencode/agent/build-reviewer.md`) with the OpenCode shape -
`mode: subagent`, a provider-prefixed `model`, and a `permission` block -
keeping the original system prompt body:

```markdown
---
description: Independent adversarial reviewer for the build loop. Read-only. Judges an implementation against its full specification bundle and returns a verdict plus actionable feedback.
mode: subagent
model: opencode-go/kimi-k2.6
permission:
  edit: deny
  bash: allow
---

(paste the system prompt body from agents/build-reviewer.md here)
```

Set `model` to whichever provider and model you run. OpenCode invokes subagents
through its own task tool, so where `build` says "spawn the `build-reviewer`
subagent", OpenCode dispatches to this agent by name - the loop behaves the same.

## Other workflows worth a look

This is simply my preferred way of working - it suits how I like to think a
feature through before any code is written, but it is one option among many. A
few others worth exploring:

- [GitHub spec-kit](https://github.com/github/spec-kit) - the spec → plan →
  tasks toolkit this flow borrows its overall shape from.
- [OpenSpec](https://github.com/Fission-AI/OpenSpec) - a spec-driven workflow
  with a stronger emphasis on persisted, version-controlled specifications.
- [Matt Pocock's AI Hero skills](https://www.aihero.dev/skills) - including the
  Grill Me skill the interview step here is based on.
- [Geoffrey Huntley's "Ralph" technique](https://ghuntley.com/ralph/) - running
  an agent in a continuous loop against a backlog.
- [Superpowers](https://github.com/obra/superpowers) - Jesse Vincent's broad
  collection of Claude Code skills for everyday development.
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) - a
  curated catalogue of Claude Code skills, commands and workflows.

## Licence

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the
full text.
