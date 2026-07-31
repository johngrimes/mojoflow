---
name: spec
description: Turn a feature idea into a complete, self-contained specification - interview the user to shared understanding, then generate spec, plan, and an executable task breakdown into a per-project specs directory. Use when the user wants to spec a feature, write a spec, or mentions "/spec".
argument-hint: "<feature description>"
---

You produce a complete specification for a feature: you interview the user to
shared understanding, then generate the spec, design plan, and a dependency-
ordered task list as files under the project's `.local/specs/` directory. The
companion `build` skill consumes these artifacts.

This skill is fully self-contained. It owns its templates (in `templates/`) and
its wireframing assets (in `wireframing/`); it does not call any external CLI or
other skill.

## Inputs

`$ARGUMENTS` is the natural-language feature description. If empty, ask the user
what they want to build before doing anything else.

## Procedure

The flow is **grill → spec → grill → plan → tasks**, run as a soft flow: there
are no hard approval gates, the user can interrupt at any point, and the second
grilling naturally surfaces anything wrong with the spec.

### 1. Grill: requirements, scope, and UI

Interview the user relentlessly about the feature - one question at a time, each
with your recommended answer - until you reach shared understanding. Walk each
branch of the decision tree, resolving dependencies between decisions in order.
When a question can be answered by exploring the codebase, explore instead of
asking. Cover at least: scope and boundaries, the prioritised user journeys and
their acceptance criteria, edge cases, data involved, success criteria, and -
crucially - whether the feature has a user interface and what screens it needs.

Do not write any files during this step. Stop grilling when the requirements are
unambiguous, not before.

### 2. Create the feature directory

Resolve the repository root (the working directory, or the nearest enclosing git
repository if there is one). Under `<repo-root>/.local/specs/`, choose the next
sequential name `NNN-short-name`:

- `NNN` is the next free 3-digit number after scanning existing directories
  (start at `001`).
- `short-name` is a 2-4 word kebab-case name you derive from the feature
  (action-noun where possible, preserving technical terms, e.g. `user-auth`,
  `oauth2-api-integration`, `fix-payment-timeout`).

Create `<repo-root>/.local/specs/NNN-short-name/`. This is the **feature
directory**; use its absolute path for all writes below.

### 3. Write the specification

Read `templates/spec-template.md` (relative to this skill) and write
`<feature-dir>/spec.md`, replacing placeholders with concrete detail from the
grilling. Keep it about WHAT and WHY, not HOW - no tech stack or code structure
here. Use a minimum of `[NEEDS CLARIFICATION]` markers; grilling should have
removed nearly all of them.

Then write a spec-quality checklist to `<feature-dir>/checklists/requirements.md`
(use `templates/checklist-template.md` for structure) with these items, and
revise the spec until they pass:

- Content quality: no implementation detail; focused on user value; mandatory
  sections complete.
- Requirement completeness: no `[NEEDS CLARIFICATION]` left; requirements
  testable and unambiguous; success criteria measurable and technology-agnostic;
  acceptance scenarios and edge cases defined; scope bounded; assumptions
  recorded.
- Feature readiness: every functional requirement has acceptance criteria; user
  scenarios cover the primary flows.

### 4. Generate wireframes (only if the feature has a UI)

If the feature involves screens, generate lo-fi HTML wireframes into
`<feature-dir>/wireframes/`, following `wireframing/ui-patterns.md` and starting
each screen from `wireframing/wireframe-template.html` (both relative to this
skill). One self-contained HTML file per screen, grayscale only, realistic
placeholder content, screens linked so the prototype is clickable. List the
screens and link the wireframes from the spec's "User Interface" section.

### 5. Grill: technical approach

Interview the user again - same relentless, one-at-a-time, recommendation-first
style - about the technical decisions: language and version, frameworks and
dependencies, storage, testing tools, target platform, project structure,
performance and scale constraints, and the shape of any external interface
contracts. Explore the existing codebase to ground these (detect the stack,
conventions, and structure already in use) rather than asking what you can
discover. Stop when the approach is settled.

### 6. Write the plan

Read `templates/plan-template.md` and write `<feature-dir>/plan.md`. Fill the
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
- `contracts/` - only if the feature exposes an interface: the contract in the
  format appropriate to the project type. Skip for purely internal work.
- `quickstart.md` - the key end-to-end integration scenarios.

### 7. Write the tasks

Read `templates/tasks-template.md` and write `<feature-dir>/tasks.md`, derived
from the spec's user stories (with priorities), the plan, the data model, and
the contracts. Rules:

- Organise by user story; each story is an independently testable increment.
- **Test-driven development is mandatory**: within every phase, test tasks
  precede their implementation tasks and are written to fail first.
- Every task uses the strict format `- [ ] T00N [P?] [Story?] Description with
exact file path`. `[P]` only for tasks on different files with no incomplete
  dependency; `[US1]` etc. only on user-story tasks.
- Include Setup and Foundational phases first, user stories in priority order,
  and a final Polish phase, plus a dependencies/execution-order section.

### 8. Consistency check

Do one automated pass over the artifacts and fix every issue you find:

- Every functional requirement maps to at least one task. Add any task that is
  missing.
- No task references an entity, contract, or screen the plan/spec never defines.
  Either add the missing definition or correct the stray reference.
- No contradiction between spec, plan, and tasks. Reconcile them to a single
  consistent statement.

Edit the relevant artifact in place, then re-run the pass and keep fixing until
it comes back clean, so no fix introduces a fresh inconsistency. Only where a fix
turns on a decision the grilling never settled (a genuine contradiction with no
clearly correct side) stop and ask the user rather than guessing.

### 9. Report

Tell the user the feature directory path, the files written, the
requirements-checklist result, any consistency issues found and how you resolved
them, and that they can proceed with `build`.

## Visibility

Announce each phase as you enter it ("Grilling on requirements", "Writing the
spec", "Grilling on the technical approach", ...). Keep the user informed of what
exists on disk after each phase.
