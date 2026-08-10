---
name: spec
description: Turn a feature idea into a complete, self-contained specification - interview the user to shared understanding, then generate spec, plan, and an executable task breakdown into a per-project specs directory. Use when the user wants to spec a feature, write a spec, or mentions "/spec".
argument-hint: "<feature description>"
---

You interview the user to shared understanding, then write the spec, design plan
and a dependency-ordered task list as files under the project's `.local/specs/`
directory for the companion `build` skill to consume. The flow is **grill → spec
→ grill → plan → tasks**, run soft: no approval gates, the user can interrupt at
any point, and the second grilling surfaces anything wrong with the spec.

This skill is self-contained. It owns its templates (`templates/`) and its
wireframing assets (`wireframing/`), and calls no external CLI or other skill.

## Where specs live

Specs always live in the **main repository**, never in a linked worktree. Resolve
`<repo-root>` as follows: run `git rev-parse --path-format=absolute --git-common-dir`
from the working directory; `<repo-root>` is the parent directory of the
resulting `.git` directory. In a normal checkout this is just the repository
root, but in a linked worktree it resolves to the main repository's root, so a
spec written from a worktree lands in the main repo's `.local/specs/` where it
remains visible after the worktree is removed. If there is no enclosing git
repository, use the working directory.

`.local/specs/` holds only the specs that are not built yet or are being built
right now; `build` moves each one into `.local/specs/archive/` once it is
implemented. So a directory sitting directly under `.local/specs/` is
outstanding work, and `archive/` is the record of what has already been done.

## Inputs

`$ARGUMENTS` is the natural-language feature description. If empty, ask the user
what they want to build before doing anything else.

## Procedure

### 1. Grill: requirements, scope and UI

Interview the user relentlessly - one question at a time, each with your
recommended answer - until the requirements are unambiguous, not before. Walk
each branch of the decision tree, resolving dependencies between decisions in
order, and explore the codebase rather than asking anything it can answer. Cover
at least: scope and boundaries, the prioritised user journeys and their
acceptance criteria, edge cases, data, success criteria, and - crucially -
whether the feature has a user interface and what screens it needs. Write no
files during this step.

### 2. Create the feature directory

Under `<repo-root>/.local/specs/` - with `<repo-root>` resolved per "Where specs
live" above (the main repository root, even when working in a linked worktree) -
create the next sequential `NNN-short-name`:

- `NNN` is one past the highest 3-digit number found across **both**
  `.local/specs/` and `.local/specs/archive/` (start at `001` when neither
  holds any). Numbers are never reused: an archived spec still consumes its
  number.
- `short-name` is a 2-4 word kebab-case name derived from the feature,
  action-noun where possible and preserving technical terms (e.g. `user-auth`,
  `oauth2-api-integration`, `fix-payment-timeout`).

This is the **feature directory**; use its absolute path for all writes below.

### 3. Write the specification

From `templates/spec-template.md` (relative to this skill), write
`<feature-dir>/spec.md`, replacing placeholders with concrete detail from the
grilling. Keep it about WHAT and WHY, not HOW - no tech stack or code structure.
Grilling should have removed nearly every `[NEEDS CLARIFICATION]` marker.

Then write a spec-quality checklist to `<feature-dir>/checklists/requirements.md`
(structure from `templates/checklist-template.md`), and revise the spec until
every item passes:

- Content quality: no implementation detail; focused on user value; mandatory
  sections complete.
- Requirement completeness: no `[NEEDS CLARIFICATION]` left; requirements
  testable and unambiguous; success criteria measurable and technology-agnostic;
  acceptance scenarios and edge cases defined; scope bounded; assumptions
  recorded.
- Feature readiness: every functional requirement has acceptance criteria; user
  scenarios cover the primary flows.

### 4. Generate wireframes (only if the feature has a UI)

Generate lo-fi HTML wireframes into `<feature-dir>/wireframes/`, following
`wireframing/ui-patterns.md` and starting each screen from
`wireframing/wireframe-template.html`. One self-contained file per screen,
grayscale only, realistic placeholder content, screens linked so the prototype
is clickable. List the screens and link the wireframes from the spec's "User
Interface" section.

### 5. Grill: technical approach

Interview the user again in the same style about the technical decisions:
language and version, frameworks and dependencies, storage, testing tools,
target platform, project structure, performance and scale constraints, and the
shape of any external interface contracts. Ground these in the existing codebase
by detecting the stack, conventions and structure already in use, rather than
asking what you can discover. Stop when the approach is settled.

### 6. Write the plan

From `templates/plan-template.md`, write `<feature-dir>/plan.md`. Fill the
Technical Context from the grilling, and complete the **Constitution Check**
against the user's CLAUDE.md (global and any project-level): confirm the design
is the simplest solution that works, that tests are planned before
implementation, and that it follows the documented language/framework
conventions. Record and justify any violation, or revise the design.

Then produce the Phase 0/1 design artifacts:

- `research.md` - one decision/rationale/alternatives entry per resolved
  technical question.
- `data-model.md` - only if the feature involves data: entities, fields,
  relationships, validation, state transitions.
- `contracts/` - only if the feature exposes an interface, in the format
  appropriate to the project type. Skip for purely internal work.
- `quickstart.md` - the key end-to-end integration scenarios.

### 7. Write the tasks

From `templates/tasks-template.md`, write `<feature-dir>/tasks.md`, derived from
the spec's user stories (with priorities), the plan, the data model and the
contracts:

- Organise by user story; each story is an independently testable increment.
- **Test-driven development is mandatory**: within every phase, test tasks
  precede their implementation tasks and are written to fail first.
- Every task uses the strict format `- [ ] T00N [P?] [Story?] Description with
exact file path`. `[P]` only for tasks on different files with no incomplete
  dependency; `[US1]` etc. only on user-story tasks.
- Setup and Foundational phases first, then user stories in priority order, then
  a Polish phase, plus a dependencies/execution-order section.

### 8. Consistency check

Pass over the artifacts and fix every issue you find:

- Every functional requirement maps to at least one task; add any task that is
  missing.
- No task references an entity, contract or screen the plan/spec never defines;
  add the missing definition or correct the stray reference.
- No contradiction between spec, plan and tasks; reconcile them to a single
  consistent statement.

Edit in place, then re-run the pass until it comes back clean, so no fix
introduces a fresh inconsistency. Only where a fix turns on a decision the
grilling never settled - a genuine contradiction with no clearly correct side -
stop and ask the user rather than guessing.

### 9. Report

Tell the user the feature directory path, the files written, the
requirements-checklist result, any consistency issues found and how you resolved
them, and that they can proceed with `build`.

## Visibility

Announce each phase as you enter it ("Grilling on requirements", "Writing the
spec", "Grilling on the technical approach", ...), keeping the user informed of
what exists on disk after each one.
