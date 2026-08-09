# MojoFlow

Four skills for an agentic coding workflow, taking a feature from a
rough idea to a finished, reviewed implementation:

- **`spec`** - interview yourself to a shared understanding, then write a
  complete, self-contained specification bundle.
- **`build`** - implement that bundle, then run an adversarial review loop until
  an independent reviewer is satisfied.
- **`freehand`** - skip the specification and build straight from a prompt, with
  the same review at the end.
- **`constitution`** - draft or amend a project's non-negotiable principles in
  `CLAUDE.md`, where the building skills then honour them.

They hand off cleanly. `spec` writes a feature directory under the project's
`.local/specs/`, which `build` consumes and then moves into
`.local/specs/archive/`. `freehand` writes nothing to disk and
hands off to nothing. `constitution` writes to `CLAUDE.md`, which the other
three pick up automatically.

The overall shape - the spec → plan → tasks progression, the artifact set, the
sequential `NNN-short-name` feature directories, and the project constitution -
draws on GitHub's [spec-kit](https://github.com/github/spec-kit).

## The workflow

### Two routes to an implementation

The choice is about how much needs settling before code is worth writing.

**`spec` → `build`** is the considered route. The interview walks the
requirements, the screens and the technical approach, and leaves a bundle on
disk that `build` then implements task by task. Reach for it when the feature is
large, when the requirements are genuinely unsettled, or when you want the
decisions reviewable before any code exists.

**`freehand`** is the direct route. It reads the codebase, asks only what it
cannot work out for itself, decides the rest and tells you what it decided, then
builds. Reach for it when the shape of the thing is already clear in your head.

Both routes end in an adversarial review loop that runs until an independent
reviewer is satisfied. `freehand` gives up the specification, not the standard.

The cost of picking wrong is asymmetric. `freehand` on something that turns out
to be large leaves you with code but no record of why it is shaped that way;
`spec` on something small costs only the interview. When torn, `spec` is the
cheaper mistake.

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
constitution/               The /constitution skill
  SKILL.md
  references/               constitution template
agents/                     The reviewer agents, one pair per coding agent
  claude/                     Claude Code (fable, medium effort)
  opencode/                   opencode (kimi-k3, high effort)
pi/                         The /build and /freehand skills for pi, which
  build/                      spawn a fresh headless pi process as the
    SKILL.md                  reviewer instead of dispatching a subagent
    build-reviewer.md          (pi has none); each owns its reviewer prompt
  freehand/
    SKILL.md
    freehand-reviewer.md
```

Each `agents/` subdirectory holds a `build-reviewer.md` and a
`freehand-reviewer.md`. The `pi/` directory holds pi-only variants of `build`
and `freehand`; see [The reviewer for pi](#the-reviewer-for-pi).

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

1. Load the spec bundle and determine the project's build, test, and lint
   commands.
2. Implement every phase in `tasks.md` order, following test-driven development,
   marking each task done as it genuinely completes and committing as the work
   reaches a green state.
3. Run the full build, test suite, and linter until green.
4. Spawn a fresh `build-reviewer` subagent that attacks the work against the
   spec.
5. Read its verdict line (`SATISFIED` ends the loop, `NEEDS_WORK` continues),
   incorporate the feedback and loop, up to the round cap (default 3).
6. On `SATISFIED`, move the feature directory into `.local/specs/archive/`.
7. Report the rounds run, final verdict, and what changed.

Each phase and each review round land as their own atomic commits, so the loop
ends with a clean working tree. The skill creates no branches and pushes
nothing.

Work is done only when the mandatory checks pass, every `tasks.md` item is
genuinely complete, and the reviewer returns `VERDICT: SATISFIED`. That is also
what triggers the archive move, so `.local/specs/` reads as a list of
outstanding work: what is in it is unbuilt or being built, and everything
finished sits in `archive/`. Numbering runs across both, so an archived spec
keeps its number for good.

The review waits for a finished `tasks.md` because that is what the
specification describes. Acceptance scenarios and success criteria are written
about a finished feature, so a reviewer sent in earlier would judge work against
a contract it is not yet meant to satisfy.

### freehand

Builds straight from a prompt, with no specification bundle and nothing written
to disk but the code:

1. Read the codebase first, so nothing discoverable on disk becomes a question.
2. Grill on requirements only, one question at a time. The default for an open
   question is to decide it rather than ask - questions are for what would
   materially change what gets built, and zero questions is a fine outcome for a
   clear prompt.
3. Implement, following test-driven development for every behaviour change.
4. Run the full build, test suite, and linter until green.
5. Spawn a fresh `freehand-reviewer` subagent that attacks the work against the
   transcript, then incorporate the feedback and loop, up to the round cap
   (default 3).
6. Report the rounds run, final verdict, what changed, every assumption it
   decided rather than asked about, and every test it skipped.

It never asks about the technical approach - language, frameworks, libraries,
file layout and test tooling all come from reading the project. Nor does it
redirect you to `spec`, however large the request turns out to be; picking the
route is yours. Like `build`, it commits as it goes and leaves a clean working
tree.

#### What the reviewer judges against

With no specification on disk, the contract is the **transcript**: your original
prompt and every interview exchange, verbatim, re-sent to the reviewer in full
each round. Verbatim is the point. A summary would be the builder's own account
of the requirements, so the reviewer would be checking the implementation
against the same understanding that produced it, and any drift or quiet
descoping would be invisible.

The assumptions and the skipped tests are declared for the same reason: a
judgement call nobody was told about cannot be reviewed. An undeclared decision
that should have been declared is itself a finding.

### The reviewer agents

Both routes end at an independent, read-only, deliberately adversarial reviewer.
It reads its contract directly rather than the orchestrator's summary of it,
proves the implementation works end to end by running it, attacks the weakest
points, and returns a machine-readable verdict plus an ordered list of required
changes. The skill spawns it fresh each round.

There are two, because they judge against different contracts: `build-reviewer`
reads a specification bundle from disk, while `freehand-reviewer` is handed a
transcript and has to weigh declared assumptions and test skips along with it.
Everything else - the adversarial stance, the mandatory prove-it-works step, the
focus areas, the verdict format - is the same by design, and duplicated rather
than shared. Nothing keeps the copies in step, so a change to a passage in one
is usually a change the others need too.

Each system prompt is identical across coding agents; only the frontmatter
differs, since each agent has its own shape and model syntax.

### The reviewer for pi

pi has no subagents and ships no Task tool, so the pi variants of `build` and
`freehand` in `pi/` do not dispatch a reviewer by name. Instead they spawn a
fresh pi process headlessly (`pi -p --no-session`), pipe the reviewer prompt
and the round's inputs to it on stdin, and read the verdict from stdout. Each
pi skill owns its reviewer prompt (`build-reviewer.md` /
`freehand-reviewer.md`), so there are no agent files to install.

The spawned reviewer starts with no session memory, inherits the project's
context files and skills (including `agent-browser` for web verification), runs
read-only (`--tools read,bash,grep,find,ls`), and ships set to
`opencode-go/kimi-k3` at high thinking; change the `--model` and `--thinking`
flags in the skill to whatever you run.

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

## Usage

```
/spec <feature description>
/build [spec-selector] [max-rounds]
/freehand <what you want built>
```

`spec` writes its artifacts under `<repo-root>/.local/specs/NNN-short-name/`,
and `build` moves that directory to `.local/specs/archive/` once the reviewer is
satisfied. `build` defaults to the latest outstanding spec when no selector is
given. `freehand` takes its whole argument as the prompt. Both default to a cap
of three review rounds, changed by asking in words ("up to 5 rounds").

## Installation

Each skill is a directory containing a `SKILL.md`; installing means placing it
under a skills directory the coding agent scans. For Claude Code and opencode,
also install the pair of reviewers under `agents/` that matches the agent you
run, or just the one whose skill you use. pi needs no agents - its reviewer
prompts live inside the pi skill variants (see [The reviewer for
pi](#the-reviewer-for-pi)).

### Claude Code

Place the `spec`, `build`, `freehand` and `constitution` directories under
`.claude/skills/<name>/` in a project, or `~/.claude/skills/<name>/` globally.
Discovered skills load automatically, with no per-skill enable step.

For the reviewers, place `agents/claude/build-reviewer.md` and
`agents/claude/freehand-reviewer.md` - already in Claude Code's agent shape, set
to `fable` at medium effort - under `.claude/agents/` (project) or
`~/.claude/agents/` (global), keeping their file names.

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
      "freehand": "allow",
      "constitution": "allow"
    }
  }
}
```

Place the reviewers from `agents/opencode/` - carrying the OpenCode shape
(`mode: subagent`, a provider-prefixed `model`, a `reasoningEffort`, and a
`permission` block) - under `.opencode/agent/` or
`~/.config/opencode/agent/`. They ship set to `opencode-go/kimi-k3` at high
effort; change `model` to whatever you run.

### pi

[pi](https://pi.dev) discovers skills under `.pi/skills/` or `.agents/skills/`
in a project (and ancestors up to the git root), or `~/.pi/agent/skills/` or
`~/.agents/skills/` globally, and loads them automatically with no per-skill
allow step.

`spec` and `constitution` work as-is. For `build` and `freehand`, use the pi
variants in `pi/` instead of the top-level pair, copying `pi/build` as `build`
and `pi/freehand` as `freehand` (only one version of each skill name can be
installed). Each pi variant is self-contained - the reviewer prompt lives
inside the skill directory - so there are no agent files to install.

On every platform the `name` and `description` frontmatter are the only required
fields; the Claude-specific `argument-hint` is ignored elsewhere. Wherever a
skill says "spawn the `build-reviewer` subagent" or "spawn the
`freehand-reviewer` subagent", Claude Code and opencode dispatch to that agent
by name. pi has no subagents, so its skill variants instead spawn a fresh pi
process to play the reviewer - see [The reviewer for pi](#the-reviewer-for-pi).

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
