---
name: constitution
description: Examine a project and draft (or amend) a constitution of non-negotiable principles, then write it into CLAUDE.md. Use when the user asks to "create a constitution", "draft project principles", "set up a constitution", "amend the constitution", or invokes "/constitution". The constitution is stored as a section of the project's CLAUDE.md file.
---

# Project constitution

Draft or amend a project's constitution - a set of non-negotiable principles
that govern how the project is specified, planned, built and reviewed. The
constitution lives as a top-level section inside the project's `CLAUDE.md`
file at the repository root, so it is loaded automatically into every Claude
Code session for the project.

## When to use

- Bootstrapping a new project that has no constitution yet.
- Amending an existing `## Constitution` section in `CLAUDE.md`.
- The user wants a single source of truth for project principles, separate
  from general coding guidance.

## Workflow

### 1. Examine the project

Before drafting anything, build a picture of the project. Read in this order
and stop once enough signal is gathered:

1. `CLAUDE.md` at the repo root (note any existing `## Constitution` section -
   this is an amendment, not a fresh draft).
2. `README.md` and any `docs/` directory.
3. `package.json` / `pyproject.toml` / `pom.xml` / `build.gradle` / `Cargo.toml`
   etc. to identify the language, framework and key dependencies.
4. Top-level source layout (`src/`, `tests/`, `infra/`, etc.) to understand
   the architecture.

Note explicitly when something is missing (e.g. no tests directory, no CI
config); these gaps often inform principles.

### 2. Confirm scope with the user

Ask only what cannot be inferred from the project. Typical questions:

- Is this a fresh constitution or an amendment to the existing one?
- How many principles (five is a reasonable default, but match what the
  project actually needs)?
- Are there non-negotiables the user wants captured verbatim (e.g. "TDD is
  mandatory", "no classes in TypeScript")?

Keep the interview tight. If the user has already given strong signals in
their request, skip the questions and draft directly.

### 3. Draft the constitution

Use the template at [references/constitution-template.md](references/constitution-template.md).
Follow these rules:

- **Principles must be declarative and testable.** A reviewer should be
  able to look at a PR and say "this violates Principle III" without
  ambiguity. Prefer MUST / SHOULD / MUST NOT over vague "should consider"
  language.
- **One concern per principle.** If a principle name contains "and", it is
  probably two principles.
- **Include a short rationale** when the reason for a rule is not obvious;
  this stops future readers from dismissing it as arbitrary.
- **No bracketed placeholders in the final output.** Replace every
  `[TOKEN]` with concrete text, or remove the section entirely.
- Headings must match the template; do not promote or demote levels within
  the constitution body.

Avoid:

- **Vague language.** "We value quality" is not a principle; "All public
  APIs MUST have integration tests" is.
- **Operational detail.** The constitution is not a runbook. "Deploy on
  Tuesdays" belongs in a deployment doc.
- **Restating language idioms.** "Use camelCase" belongs in a style guide,
  not the constitution.
- **Unfalsifiable principles.** If a PR cannot violate it, it is not a
  principle.

### 4. Place the constitution into CLAUDE.md

The constitution lives under a single `## Constitution` heading at the top
level of `CLAUDE.md`. Inside that section the constitution body retains its
own heading hierarchy - but demoted by one level so the document remains
well-formed (i.e. template `## Core Principles` becomes `### Core Principles`
inside CLAUDE.md, and `### Principle Name` becomes `#### Principle Name`).

Placement rules:

- If `CLAUDE.md` does not exist, create it with the constitution as the only
  section.
- If `CLAUDE.md` exists but has no `## Constitution` section, insert the new
  section immediately after the file's introductory paragraph (or at the
  top if there is none) and before any other top-level sections.
- If `CLAUDE.md` already has a `## Constitution` section, replace it
  entirely with the new content. Do not try to merge in place - the user
  can review the diff.

Always use the Edit or Write tool against `CLAUDE.md` directly. Do not
create a separate constitution file.

### 5. Report back

After writing, summarise for the user:

- Files modified (just `CLAUDE.md` in the common case).
- A one-line description of what was added or changed.
- A suggested commit message, e.g. `Adopt project constitution` or
  `Amend constitution to add Observability principle`.

Do not commit the change unless the user asks.

## Notes

- Keep the constitution short. Five principles is the sweet spot; more than
  seven and it stops being read.
- The constitution is for project-specific non-negotiables. General coding
  guidance (formatting, naming conventions, language idioms) belongs
  elsewhere in `CLAUDE.md` or in dedicated rules files - not in the
  constitution.
