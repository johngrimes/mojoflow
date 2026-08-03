# MojoFlow

Four Claude Code skills for an agentic coding workflow, taking a feature from a
rough idea to a finished, reviewed implementation:

- **`spec`** - interview yourself to a shared understanding, then write a
  complete, self-contained specification bundle.
- **`build`** - implement that bundle a phase at a time, each phase closing with
  an adversarial build-and-review loop that runs until an independent reviewer
  is satisfied.
- **`freehand`** - skip the specification and build straight from a prompt, with
  the same adversarial review at the end.
- **`constitution`** - draft or amend a project's non-negotiable principles and
  write them into `CLAUDE.md`, where the building skills then honour them.

The skills hand off cleanly. `spec` writes a feature directory under the
project's `.local/specs/`, which `build` consumes. `freehand` writes nothing to
disk and hands off to nothing - it is that whole route in a single command.
`constitution` writes to `CLAUDE.md`, which `spec` (during its constitution
check), `build` and `freehand` all pick up automatically.

The overall shape - the spec → plan → tasks progression, the artifact set
(`spec.md`, `plan.md`, `tasks.md`, `research.md`, `data-model.md`, `contracts/`,
`quickstart.md`), the sequential `NNN-short-name` feature directories, and the
project constitution - draws on GitHub's
[spec-kit](https://github.com/github/spec-kit).

## The workflow

### Two routes to an implementation

The choice between them is about how much needs settling before code is worth
writing.

**`spec` → `build`** is the considered route. The interview walks the
requirements, the screens and the technical approach, and leaves a bundle on
disk that `build` then implements a phase at a time. Reach for it when the
feature is large or touches several parts of the system, when the requirements
are genuinely unsettled, or when you want the decisions written down and
reviewable before any code exists.

**`freehand`** is the direct route. It reads the codebase, asks only what it
cannot work out for itself, decides the rest and tells you what it decided, then
builds. Nothing lands on disk but the code. Reach for it when the shape of the
thing is already clear in your head and a specification bundle would be ceremony
around a decision you have already made.

Both routes end the same way, in an adversarial review loop that runs until an
independent reviewer is satisfied - `build` at the close of every phase,
`freehand` once at the end. The rigour lives in the review either way -
`freehand` gives up the specification, not the standard.

If you pick wrong, the cost is small in one direction and not the other. Running
`freehand` on something that turns out to be large leaves you with code but no
record of why it is shaped that way; running `spec` on something small costs
only the interview. When genuinely torn, `spec` is the cheaper mistake.

### Where to start

**A new project.** Lead with a `spec` for the first slice of work, so the shape
of the thing becomes concrete. Once there is enough to reason about, draft the
`constitution` - the early spec gives you real material to ground the principles
in, rather than guessing them up front. From then on the rhythm is a route per
feature, with the constitution steering whichever one you take and amended
whenever a new principle earns its place.

**An existing project.** Start with the `constitution`. The code, docs and build
files already encode how the project works, so the skill reads them and distils
the principles already in force. Each new piece of work then takes whichever
route suits it, anchored to a constitution that reflects the project as it
actually is.

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
freehand/                   The /freehand skill
  SKILL.md
constitution/               The /constitution skill
  SKILL.md
  references/               constitution template
agents/                     The reviewer agents, one pair per coding agent
  claude/                     Claude Code (fable, medium effort)
    build-reviewer.md
    freehand-reviewer.md
  opencode/                   opencode (glm-5.2, high effort)
    build-reviewer.md
    freehand-reviewer.md
  pi/                         pi (glm-5.2, high effort)
    build-reviewer.md
    freehand-reviewer.md
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

It follows that the resulting spec is written for the agent, not a human
reader - a precise, machine-consumable record of the decisions reached, and the
input `build` works from. There is little value in committing these specs: they
are an artifact of a particular agent session, not source. They live under
`.local/`, ignored by Git globally, and stay out of version control by design.

The grilling idea comes from Matt Pocock's
[Grill Me skill](https://www.aihero.dev/skills-grill-me).

### build

Implements a feature from the `spec` bundle, then runs an adversarial review
loop. Where `tasks.md` has more than one phase, an invocation builds **one
phase** and stops; you run it again to take the next:

1. Load the spec bundle and determine the project's build, test, and lint
   commands.
2. Resolve the phase to build - the next one in `tasks.md` with unfinished
   tasks.
3. Implement that phase and no other, following test-driven development, marking
   each task done as it genuinely completes and committing as the work reaches a
   green state.
4. Run the full build, test suite, and linter until green.
5. Spawn a fresh `build-reviewer` subagent that attacks the phase against the
   spec.
6. Read its verdict line: `SATISFIED` ends the loop; `NEEDS_WORK` continues.
7. Incorporate the feedback and loop, up to the round cap (default 3).
8. Report the phase built, the rounds run, the final verdict, what changed, and
   the phases still outstanding.

The skill commits as it goes - the phase and each review round land as their own
atomic commits - so the loop ends with a clean working tree. It does not create
branches or push to any remote; that stays with you.

A phase is done only when the mandatory checks pass, every task in it is
genuinely complete, and the reviewer returns `VERDICT: SATISFIED`.

#### Why one phase at a time

A whole feature is more than a reviewer can attack in depth in a single pass.
The diff is large, the round cap gets spent on the loudest findings, and the
quieter ones ride through underneath them. A phase is small enough to be run,
exercised and genuinely picked apart within the cap.

It is also where you get a say. Each invocation ends at a reviewed, committed
increment you can inspect - and correct, or redirect - before any of the next
phase is written on top of it.

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
   transcript.
6. Read its verdict line, incorporate the feedback and loop, up to the round cap
   (default 3).
7. Report the rounds run, final verdict, what changed, every assumption it
   decided rather than asked about, and every test it skipped.

It never asks about the technical approach - language, frameworks, libraries,
file layout and test tooling all come from reading the project. Nor does it
redirect you to `spec`, however large the request turns out to be; picking the
route is yours. Like `build`, it commits as it goes and leaves a clean working
tree; it creates no branches and pushes nothing.

#### What the reviewer judges against

With no specification on disk, the contract is the **transcript**: your original
prompt and every interview exchange, verbatim, re-sent to the reviewer in full
each round. Verbatim is the point. A summary would be the builder's own account
of the requirements, so the reviewer would be checking the implementation
against the same understanding that produced it, and any drift or quiet
descoping between the interview and the code would be invisible. The transcript
is the one statement of intent the builder cannot revise.

The assumptions and the skipped tests are declared to the reviewer for the same
reason: a judgement call nobody was told about cannot be reviewed. The reviewer
weighs them against the transcript, and an undeclared decision that should have
been declared is itself a finding.

### The reviewer agents

Both routes end at an independent, read-only, deliberately adversarial reviewer.
It reads its contract directly rather than the orchestrator's summary of it,
proves the implementation works end to end by running it, attacks the weakest
points, and returns a machine-readable verdict plus an ordered list of required
changes. The skill spawns it fresh each round.

There are two, because they judge against different contracts: `build-reviewer`
reads a specification bundle from disk and judges the one phase under review,
treating later phases as legitimately unbuilt, while `freehand-reviewer` is
handed a transcript and has to weigh declared assumptions and test skips along
with it.
Everything else - the adversarial stance, the mandatory prove-it-works step, the
focus areas, the verdict format - is the same by design, and duplicated rather
than shared. Nothing keeps the copies in step, so a change to a passage in one
is usually a change the others need too.

Each system prompt is identical across coding agents; only the frontmatter
differs, since each agent has its own shape and model syntax. `agents/`
therefore holds a pair of definitions per platform - `claude/`, `opencode/` and
`pi/` - and you install the pair that matches the agent you run.

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

Because the constitution lives in `CLAUDE.md`, `spec` (which runs a constitution
check while planning), `build` and `freehand` all pick it up without any extra
wiring.

## Usage

```
/spec <feature description>
/build [spec-selector] [max-rounds]
/freehand <what you want built>
```

`spec` writes its artifacts under `<repo-root>/.local/specs/NNN-short-name/`.
`build` defaults to the latest spec when no selector is given, and within it to
the next phase with unfinished tasks; name a different phase in words ("build
phase 4") to override that. `freehand` takes
its whole argument as the prompt and writes nothing to disk. Both default to a
cap of three review rounds, changed by asking in words ("up to 5 rounds").

## Installation

Each skill is a directory containing a `SKILL.md`; installing means placing it
under a skills directory the coding agent scans. Each build loop then spawns its
reviewer subagent - `build-reviewer` or `freehand-reviewer` - whose definitions
differ per platform. Install the pair under `agents/` that matches the agent you
run, or just the one whose skill you use.

### Claude Code

Place the `spec`, `build`, `freehand` and `constitution` directories under a
skills directory Claude Code reads - `.claude/skills/<name>/` in a project, or
`~/.claude/skills/<name>/` globally. Claude Code loads discovered skills
automatically, with no per-skill enable step.

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

For the reviewers, `agents/opencode/build-reviewer.md` and
`agents/opencode/freehand-reviewer.md` carry the OpenCode shape
(`mode: subagent`, a provider-prefixed `model`, a `reasoningEffort`, and a
`permission` block) with the same system prompt bodies. Place them under
`.opencode/agent/`
(or `~/.config/opencode/agent/`). They ship set to `opencode-go/glm-5.2` at high
effort; change `model` to whatever you run.

### pi

[pi](https://pi.dev) discovers skills under its own paths - `.pi/skills/` or
`.agents/skills/` in a project (and ancestors up to the git root), or
`~/.pi/agent/skills/` or `~/.agents/skills/` globally. It loads discovered
skills automatically, with no per-skill allow step.

For the reviewers, pi discovers subagents from `.pi/agents/*.md` (project) or
`~/.pi/agent/agents/*.md` (global), with project definitions winning on a name
clash. Place `agents/pi/build-reviewer.md` and `agents/pi/freehand-reviewer.md`
at one of those paths. Both are already in pi's agent shape, set to
`opencode-go/glm-5.2` at high thinking; change `model` to whatever you run.

On every platform the `name` and `description` frontmatter are the only required
fields; the Claude-specific `argument-hint` is ignored elsewhere. Wherever a
skill says "spawn the `build-reviewer` subagent" or "spawn the
`freehand-reviewer` subagent", each platform dispatches to that agent by name
and the loop behaves the same.

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
