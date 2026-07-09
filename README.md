# MojoFlow

Three Claude Code skills for an agentic coding workflow, taking a feature from a
rough idea to a finished, reviewed implementation:

- **`spec`** - interview yourself to a shared understanding, then write a
  complete, self-contained specification bundle.
- **`build`** - implement that bundle, then run an adversarial build-and-review
  loop until an independent reviewer is satisfied.
- **`constitution`** - draft or amend a project's non-negotiable principles and
  write them into `CLAUDE.md`, where `spec` and `build` then honour them.

The skills hand off cleanly. `spec` writes a feature directory under the
project's `.local/specs/`, which `build` consumes. `constitution` writes to
`CLAUDE.md`, which both `spec` (during its constitution check) and `build` pick
up automatically.

The overall shape - the spec → plan → tasks progression, the artifact set
(`spec.md`, `plan.md`, `tasks.md`, `research.md`, `data-model.md`, `contracts/`,
`quickstart.md`), the sequential `NNN-short-name` feature directories, and the
project constitution - draws on GitHub's
[spec-kit](https://github.com/github/spec-kit).

## The workflow

The three skills combine differently depending on where you start.

**A new project.** Lead with a `spec` for the first slice of work, so the shape
of the thing becomes concrete. Once there is enough to reason about, draft the
`constitution` - the early spec gives you real material to ground the principles
in, rather than guessing them up front. From then on the rhythm is `spec` →
`build`, repeated per feature, with the constitution steering both and amended
whenever a new principle earns its place.

**An existing project.** Start with the `constitution`. The code, docs and build
files already encode how the project works, so the skill reads them and distils
the principles already in force. Each new piece of work then follows the same
`spec` → `build` rhythm, anchored to a constitution that reflects the project as
it actually is.

Either way, treat the constitution as a living document. As a new non-negotiable
emerges or an old assumption stops holding, amend it so it keeps describing how
the project really works. `constitution` handles amendments as well as fresh
drafts.

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
agents/                     The build-reviewer agent, one per coding agent
  claude/build-reviewer.md    Claude Code (fable, medium effort)
  opencode/build-reviewer.md  opencode (glm-5.2, high effort)
  pi/build-reviewer.md        pi (glm-5.2, high effort)
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
  `.local/`, ignored by Git globally, and stay out of version control by design.

The grilling idea comes from Matt Pocock's
[Grill Me skill](https://www.aihero.dev/skills-grill-me).

### build

Implements a feature from the `spec` bundle, then runs an adversarial review
loop:

1. Load the spec bundle and determine the project's build, test, and lint
   commands.
2. Implement every phase in `tasks.md` order, following test-driven development,
   marking each task done as it genuinely completes and committing each phase as
   an atomic unit once its tests pass.
3. Run the full build, test suite, and linter until green.
4. Spawn a fresh `build-reviewer` subagent that attacks the work against the
   spec.
5. Read its verdict line: `SATISFIED` ends the loop; `NEEDS_WORK` continues.
6. Incorporate the feedback and loop, up to the round cap (default 3).
7. Report the rounds run, final verdict, and what changed.

The skill commits as it goes - each phase and each review round lands as its own
atomic commit - so the loop ends with a clean working tree. It does not create
branches or push to any remote; that stays with you.

Work is done only when the mandatory checks pass, every `tasks.md` item is
genuinely complete, and the reviewer returns `VERDICT: SATISFIED`.

### build-reviewer agent

An independent, read-only, deliberately adversarial reviewer. It reads the
specification bundle directly (never the orchestrator's summary), proves the
implementation works end to end by running it, attacks the weakest points, and
returns a machine-readable verdict plus an ordered list of required changes.
`build` spawns it fresh each round.

The system prompt is identical across coding agents; only the frontmatter
differs, since each agent has its own shape and model syntax. `agents/`
therefore holds one definition per platform - `claude/`, `opencode/` and `pi/` -
and you install whichever matches the agent you run.

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

## Usage

```
/spec <feature description>
/build [spec-selector] [max-rounds]
```

`spec` writes its artifacts under `<repo-root>/.local/specs/NNN-short-name/`.
`build` defaults to the latest spec when no selector is given, and to a cap of
three review rounds.

## Installation

Each skill is a directory containing a `SKILL.md`; installing means placing it
under a skills directory the coding agent scans. The `build` loop then spawns a
`build-reviewer` subagent, whose definition differs per platform - install the
one under `agents/` that matches the agent you run.

### Claude Code

Place the `spec`, `build` and `constitution` directories under a skills
directory Claude Code reads - `.claude/skills/<name>/` in a project, or
`~/.claude/skills/<name>/` globally. Claude Code loads discovered skills
automatically, with no per-skill enable step.

For the reviewer, place `agents/claude/build-reviewer.md` - already in Claude
Code's agent shape, set to `fable` at medium effort - at
`.claude/agents/build-reviewer.md` (project) or `~/.claude/agents/build-reviewer.md`
(global).

### OpenCode

[OpenCode](https://opencode.ai) discovers skills by walking up from the working
directory to the git root, scanning (among others) the same
`.claude/skills/<name>/SKILL.md` locations Claude Code uses - so skills already
installed for Claude Code are found with no further work. Otherwise place them
under `.opencode/skills/<name>/` (project) or
`~/.config/opencode/skills/<name>/` (global).

Skills are gated by pattern in `opencode.json`. Allow them explicitly (values
are `allow`, `deny`, or `ask`, and patterns support wildcards):

```json
{
  "permission": {
    "skill": {
      "spec": "allow",
      "build": "allow",
      "constitution": "allow"
    }
  }
}
```

For the reviewer, `agents/opencode/build-reviewer.md` carries the OpenCode shape

- `mode: subagent`, a provider-prefixed `model`, a `reasoningEffort`, and a
  `permission` block - with the same system prompt body. Place it at
  `.opencode/agent/build-reviewer.md` (or
  `~/.config/opencode/agent/build-reviewer.md`). It ships set to
  `opencode-go/glm-5.2` at high effort; change `model` to whatever you run.

### pi

[pi](https://pi.dev) discovers skills under its own paths - `.pi/skills/` or
`.agents/skills/` in a project (and ancestors up to the git root), or
`~/.pi/agent/skills/` or `~/.agents/skills/` globally. It loads discovered
skills automatically, with no per-skill allow step.

For the reviewer, pi discovers subagents from `.pi/agents/*.md` (project) or
`~/.pi/agent/agents/*.md` (global), with project definitions winning on a name
clash. Place `agents/pi/build-reviewer.md` - already in pi's agent shape, set to
`opencode-go/glm-5.2` at high thinking - at one of those paths, changing `model`
to whatever you run.

On every platform the `name` and `description` frontmatter are the only required
fields; the Claude-specific `argument-hint` is ignored elsewhere. Wherever
`build` says "spawn the `build-reviewer` subagent", each platform dispatches to
this agent by name and the loop behaves the same.

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
