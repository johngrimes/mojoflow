# Implementation Plan: [FEATURE]

**Feature**: `[NNN-short-name]` | **Date**: [DATE] | **Spec**: [link to spec.md]

## Summary

[Primary requirement from the spec plus the chosen technical approach.]

## Technical Context

**Language/Version**: [e.g. TypeScript 5.x, Python 3.13, Java 21]
**Primary Dependencies**: [e.g. React + Vite, FastAPI, Spring Boot]
**Storage**: [e.g. PostgreSQL, files, or N/A]
**Testing**: [e.g. Vitest + Playwright, pytest, JUnit]
**Target Platform**: [e.g. Linux server, browser, iOS 15+]
**Project Type**: [e.g. library / cli / web-service / web-app / mobile-app]
**Performance Goals**: [domain-specific, or N/A]
**Constraints**: [domain-specific, e.g. <200ms p95, offline-capable, or N/A]
**Scale/Scope**: [domain-specific, e.g. 10k users, 50 screens, or N/A]

## Constitution Check

_GATE: the constitution is the user's CLAUDE.md (global and any project-level).
Re-check after the design is drafted._

Confirm the design honours the standing principles, for example:

- **Simplicity**: is this the simplest solution that meets the requirements? Is
  any abstraction, dependency, or layer not strictly necessary?
- **Test-driven development**: are tests planned before implementation for every
  behaviour? (Mandatory.)
- **Language/framework rules**: does the approach follow the documented
  conventions (e.g. functional TypeScript, `uv` for Python, Vite for React)?
- **Visibility of system status**: does every user-initiated action surface
  timely feedback?

List any violation and its justification below; if a violation cannot be
justified, revise the design rather than proceed.

| Violation | Why needed | Simpler alternative rejected because |
| --------- | ---------- | ------------------------------------ |
|           |            |                                      |

## Project Structure

### Documentation (this feature)

```text
.local/specs/[NNN-short-name]/
├── spec.md
├── plan.md            # This file.
├── research.md        # Phase 0 output.
├── data-model.md      # Phase 1 output (if the feature involves data).
├── quickstart.md      # Phase 1 output.
├── contracts/         # Phase 1 output (if the feature exposes interfaces).
├── wireframes/        # Lo-fi HTML wireframes (if the feature has a UI).
├── checklists/        # Quality checklists.
└── tasks.md           # Generated last.
```

### Source code (repository root)

<!--
  Replace the placeholder tree with the concrete layout for this feature. Show
  real paths. Remove unused branches.
-->

```text
src/
tests/
```

**Structure Decision**: [Document the selected structure and reference the real directories above.]

## Phase 0: Research

Resolve every open technical question into a decision recorded in `research.md`
using the format:

- **Decision**: [what was chosen]
- **Rationale**: [why]
- **Alternatives considered**: [what else was evaluated and why rejected]

## Phase 1: Design & Contracts

- **Data model** (`data-model.md`, if data is involved): entities, fields,
  relationships, validation rules, state transitions.
- **Interface contracts** (`contracts/`, if the feature exposes interfaces):
  the contract format appropriate to the project type (REST/GraphQL schema for
  services, command schema for CLIs, public API for libraries, etc.). Skip for
  purely internal work.
- **Quickstart** (`quickstart.md`): the key integration scenarios that exercise
  the feature end to end, phrased so they can later be validated.
