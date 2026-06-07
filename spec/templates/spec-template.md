# Feature Specification: [FEATURE NAME]

**Feature**: `[NNN-short-name]`
**Created**: [DATE]
**Status**: Draft

## User Scenarios & Testing _(mandatory)_

<!--
  User stories are PRIORITISED user journeys ordered by importance. Each story
  must be INDEPENDENTLY TESTABLE: implementing just one still yields a viable
  increment that delivers value. Assign priorities (P1, P2, P3...), P1 most
  critical. Each story should be independently developed, tested, and
  demonstrated.
-->

### User Story 1 - [Brief Title] (Priority: P1)

[Describe this user journey in plain language.]

**Why this priority**: [Value and reasoning for the priority level.]

**Independent Test**: [How this story can be verified on its own.]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome].
2. **Given** [initial state], **When** [action], **Then** [expected outcome].

---

### User Story 2 - [Brief Title] (Priority: P2)

[Describe this user journey in plain language.]

**Why this priority**: [Value and reasoning for the priority level.]

**Independent Test**: [How this story can be verified on its own.]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome].

---

[Add more user stories as needed, each with an assigned priority.]

### Edge Cases

- What happens when [boundary condition]?
- How does the system handle [error scenario]?

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST [specific capability].
- **FR-002**: System MUST [specific capability].
- **FR-003**: Users MUST be able to [key interaction].

_Mark genuinely unresolved points (kept to a minimum after grilling) as:_

- **FR-00N**: System MUST [capability] [NEEDS CLARIFICATION: specific question].

### Key Entities _(include if the feature involves data)_

- **[Entity 1]**: [What it represents, key attributes without implementation detail.]
- **[Entity 2]**: [What it represents, relationships to other entities.]

## Success Criteria _(mandatory)_

<!--
  Measurable, technology-agnostic outcomes verifiable without implementation
  detail. Include both quantitative metrics and qualitative measures.
-->

### Measurable Outcomes

- **SC-001**: [Measurable metric, e.g. "Users complete account creation in under 2 minutes".]
- **SC-002**: [Measurable metric, e.g. "System handles 1000 concurrent users without degradation".]

## User Interface _(include only if the feature has a UI)_

<!--
  When the feature involves screens, list them here and link the lo-fi HTML
  wireframes generated into the wireframes/ subdirectory of this feature.
-->

- **[Screen name]**: [Purpose and key elements.] Wireframe: `wireframes/[screen].html`

## Assumptions

<!--
  Reasonable defaults chosen where the description did not specify a detail.
-->

- [Assumption about target users, scope boundaries, data, or environment.]
- [Dependency on an existing system or service.]
