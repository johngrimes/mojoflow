# MojoFlow

Five agent-agnostic skills for an agentic coding workflow, taking a feature
from a rough idea to a finished, reviewed implementation:

- **`spec`** - interview to a shared understanding, then write a complete,
  self-contained specification bundle.
- **`build`** - implement that bundle, then run an adversarial review loop until
  an independent reviewer is satisfied.
- **`freehand`** - skip the specification and build straight from a prompt, with
  the same review at the end.
- **`quest`** - pursue an objective in a loop, delegating each attempt to a
  fresh subagent and judging the result yourself until it is genuinely done.
- **`constitution`** - draft or amend a project's non-negotiable principles in
  `CLAUDE.md`, where the building skills then honour them.

Each skill is a single self-contained `SKILL.md` with no agent-specific wiring:
no agent files, no per-agent variants. The reviewer prompts live inside the
skills, and the model and effort for each spawned subagent are listed in the
skill itself.

## The workflow

### Three routes to an implementation

The choice is about how much needs settling before code is worth writing.

**`spec` → `build`** is the considered route. The interview walks the
requirements, the screens and the technical approach, and leaves a bundle on
disk that `build` then implements phase by phase. Reach for it when the feature
is large, when the requirements are genuinely unsettled, or when you want the
decisions reviewable before any code exists.

**`freehand`** is the direct route. It grills on requirements, decides the rest
for itself, then builds. Reach for it when the shape of the thing is already
clear in your head.

Both routes end in an adversarial review loop that runs until an independent
reviewer is satisfied. `freehand` gives up the specification, not the standard.

The cost of picking wrong is asymmetric. `freehand` on something that turns out
to be large leaves you with code but no record of why it is shaped that way;
`spec` on something small costs only the interview. When torn, `spec` is the
cheaper mistake.

**`quest`** is the persistence route. It is for when the objective is clear but
the path is not - a gnarly bug, a flaky integration, a long campaign of small
fixes. You delegate the objective verbatim to a fresh subagent, verify the
result yourself, and respawn until it is genuinely achieved. No specification,
no interview, no handoff - just the loop.

### Where to start

**A new project.** Lead with a `spec` for the first slice of work, so the shape
of the thing becomes concrete. Then draft the `constitution` - the early spec
gives you real material to ground the principles in, rather than guessing them
up front. From then on, a route per feature.

**An existing project.** Start with the `constitution`. The code, docs and build
files already encode how the project works, so the skill reads them and distils
the principles already in force.

Either way, treat the constitution as a living document, amended as a new
non-negotiable emerges or an old assumption stops holding.

## What's here

```
spec/                       The /spec skill
  SKILL.md
  templates/                spec, plan, tasks, and checklist templates
  wireframing/              lo-fi HTML wireframe template and UI patterns
build/                      The /build skill
  SKILL.md
freehand/                   The /freehand skill
  SKILL.md
quest/                      The /quest skill
  SKILL.md
constitution/               The /constitution skill
  SKILL.md
  references/               constitution template
```

Each skill is self-contained: the reviewer prompts for `build` and `freehand`
live inside their `SKILL.md`, so there are no agent files to install anywhere.

## The skills

### spec

Turns a feature idea into a complete specification. The flow is
**grill → spec → grill → plan → tasks**, run as a soft flow with no hard
approval gates:

1. **Grill** on requirements, scope, and UI - one question at a time, each with
   a recommended answer - until the requirements are unambiguous.
2. Create the next sequential feature directory `NNN-short-name` under
   `<repo-root>/.local/specs/`, numbered one past the highest number in either
   `.local/specs/` or `.local/specs/archive/`.
3. Write `spec.md` (what and why, not how) plus a requirements checklist.
4. Generate lo-fi HTML wireframes into `wireframes/`, if the feature has a UI.
5. **Grill** on the technical approach - stack, frameworks, storage, testing,
   structure, interface contracts.
6. Write `plan.md` with a constitution check, plus `research.md`,
   `data-model.md`, `contracts/`, and `quickstart.md` where relevant.
7. Write a dependency-ordered, test-first `tasks.md`.
8. Run a consistency check across the artifacts, fixing what it surfaces.

The skill is self-contained: it owns its templates and wireframing assets and
calls no external CLI or other skill.

#### Why grill instead of just reading a spec

Reading a large specification is an unreliable way to build shared
understanding. A long document invites skimming, lets ambiguity hide in prose,
and gives no signal about which parts were actually absorbed. The interview
replaces that with a focused exchange: one question at a time, each forcing a
concrete decision.

It follows that the resulting spec is written for the agent, not a human
reader - a machine-consumable record of the decisions reached, and the input
`build` works from. These specs are an artifact of a particular agent session,
not source, so they live under `.local/`, ignored by Git globally, and stay out
of version control by design.

The grilling idea comes from Matt Pocock's
[Grill Me skill](https://www.aihero.dev/skills-grill-me).

### build

Implements a feature from the `spec` bundle, then runs an adversarial review
loop:

1. Delegate implementation to the `quest` skill, which invokes implementation
   agents one phase at a time - each gets the full spec but is asked for a
   single phase, so the effort stays bounded.
2. When every phase is done and every task in `tasks.md` is marked complete, run
   up to `[max-rounds]` rounds of adversarial review: spawn a fresh reviewer
   subagent that attacks the work against the spec, incorporate its feedback,
   and loop.
3. Move the feature directory into `.local/specs/archive/` once built, so
   `.local/specs/` reads as a list of outstanding work.
4. With your approval, push the branch, open a draft pull request, and watch CI
   through to green - fixing and pushing again on failure.

The review waits for a finished `tasks.md` because that is what the
specification describes. Acceptance scenarios and success criteria are written
about a finished feature, so a reviewer sent in earlier would judge work against
a contract it is not yet meant to satisfy.

### freehand

Builds straight from a prompt, with no specification bundle and nothing written
to disk but the code:

1. Refine the prompt with the `grill-me` skill, resolving ambiguities from a
   requirements perspective only.
2. Implement, following test-driven development for every behaviour change.
3. Run the full build, test suite, and linter until green.
4. Spawn a fresh reviewer subagent that attacks the work against the prompt and
   the grilling session, incorporate its feedback, and loop, up to the round
   cap.
5. With your approval, push the branch, open a draft pull request, and watch CI
   through to green - fixing and pushing again on failure.

It never asks about the technical approach - language, frameworks, libraries,
file layout and test tooling all come from reading the project. Nor does it
redirect you to `spec`, however large the request turns out to be; picking the
route is yours.

### quest

Pursues an objective in a loop until it is genuinely achieved, with no
specification and no interview:

1. Delegate the objective verbatim to a fresh subagent.
2. Judge the result yourself - run its tests, inspect what it changed, exercise
   the outcome - never trust its claim.
3. On success, stop and report. Otherwise respawn with the same objective;
   attempts are uncapped.

Each attempt is a fresh subagent with its own context, working in the same
directory, so later attempts build on earlier ones. The loop is for objectives
that are clear but hard - a bug that resists diagnosis, a flaky integration, a
long campaign of small fixes - or for "keep trying until it works". It writes
nothing to disk but the work.

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
   it loads into every session automatically - and so `spec` (which runs a
   constitution check while planning), `build` and `freehand` all pick it up
   with no extra wiring.

### The reviewers

`build` and `freehand` each end at an independent, deliberately adversarial
reviewer. It reads its contract directly rather than the orchestrator's summary
of it, proves the implementation works end to end by running it, attacks the
weakest points, and returns a machine-readable verdict plus an ordered list of
required changes. The skill spawns it fresh each round.

The two reviewer prompts live inside `build/SKILL.md` and `freehand/SKILL.md`
and differ only in their inputs: `build`'s reviewer reads a specification
bundle from disk, `freehand`'s is handed the prompt and the grilling session.
Everything else - the adversarial stance, the mandatory prove-it-works step, the
focus areas, the verdict format - is the same by design, and duplicated rather
than shared.

`quest` inverts the shape: the orchestrator judges each attempt itself, so
nothing adversarial is needed. What `quest` spawns is an implementation agent
that executes the delegated objective in the shared working directory and
reports back what it did and the state it left.

## Usage

```
/spec <feature description>
/build [spec-selector] [max-rounds]
/freehand <what you want built>
/quest <objective>
```

`spec` writes its artifacts under `<repo-root>/.local/specs/NNN-short-name/`,
and `build` moves that directory to `.local/specs/archive/` once built. `build`
defaults to the latest outstanding spec when no selector is given, and to three
review rounds unless changed in words ("up to 5 rounds"). `freehand` and `quest`
take their whole argument as the prompt or objective; `quest` attempts are
uncapped.

## Installation

The easiest way is the [skills CLI](https://skills.sh):

```
skills add johngrimes/mojoflow
```

Or install by hand: each skill is a directory containing a `SKILL.md`; copy
them under a skills directory the coding agent scans - `.claude/skills/` for
Claude Code, `.opencode/skills/` for opencode, or `.pi/skills/` for pi, in the
project or the user's home directory. There are no agent files and no
per-agent variants to install.

The `spec` and `freehand` skills use the
[grill-me](https://www.aihero.dev/skills-grill-me) skill. Install that using
this command:

```
skills add mattpocock/skills/grill-me
```

## Other workflows worth a look

This is simply my preferred way of working, and one option among many. A few
others worth exploring:

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
