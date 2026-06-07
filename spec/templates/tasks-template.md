# Tasks: [FEATURE NAME]

**Input**: Design documents in `.local/specs/[NNN-short-name]/`
**Prerequisites**: plan.md (required), spec.md (required for user stories); research.md, data-model.md, contracts/ where present.

**Tests**: Test-driven development is mandatory. Every behaviour is covered by a
test task that precedes its implementation task, and those tests must be written
to fail first.

**Organisation**: Tasks are grouped by user story so each story can be
implemented and tested independently.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: can run in parallel (different files, no dependency on incomplete tasks).
- **[Story]**: the user story a task belongs to (US1, US2, ...); omitted for Setup, Foundational, and Polish.
- Every task includes an exact file path.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialisation and basic structure.

- [ ] T001 [Setup task with file path].

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST complete before any user story begins.

- [ ] T00N [Foundational task with file path].

**Checkpoint**: Foundation ready - user story implementation can begin.

---

## Phase 3: User Story 1 - [Title] (Priority: P1) 🎯 MVP

**Goal**: [What this story delivers.]

**Independent Test**: [How to verify this story works on its own.]

### Tests for User Story 1 ⚠️ write first, confirm failing before implementing

- [ ] T00N [P] [US1] [Test for the behaviour] in tests/[path].
- [ ] T00N [P] [US1] [Test for the behaviour] in tests/[path].

### Implementation for User Story 1

- [ ] T00N [P] [US1] [Model/component] in src/[path].
- [ ] T00N [US1] [Service/endpoint] in src/[path].

**Checkpoint**: User Story 1 fully functional and independently testable.

---

## Phase 4: User Story 2 - [Title] (Priority: P2)

**Goal**: [What this story delivers.]

**Independent Test**: [How to verify this story works on its own.]

### Tests for User Story 2 ⚠️ write first, confirm failing before implementing

- [ ] T00N [P] [US2] [Test for the behaviour] in tests/[path].

### Implementation for User Story 2

- [ ] T00N [US2] [Task with file path].

**Checkpoint**: User Stories 1 and 2 both work independently.

---

[Add further user story phases following the same pattern.]

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Improvements spanning multiple user stories.

- [ ] T0NN [P] Documentation updates in docs/.
- [ ] T0NN Code cleanup and refactoring.
- [ ] T0NN Run quickstart.md validation.

---

## Dependencies & Execution Order

- **Setup (Phase 1)**: no dependencies.
- **Foundational (Phase 2)**: depends on Setup; blocks all user stories.
- **User Stories (Phase 3+)**: depend on Foundational; otherwise independent, runnable in priority order (P1 → P2 → P3) or parallel.
- **Polish (final)**: depends on the targeted user stories being complete.

Within each story: tests written and failing → models → services → endpoints →
integration. Tasks touching the same file run sequentially; `[P]` tasks on
different files may run together.

## Implementation Strategy

MVP first: Setup → Foundational → User Story 1, then validate independently
before adding further stories incrementally.
