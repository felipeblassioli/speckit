# Spec: [SHORT NAME]

**Feature ID**: [feature-id]
**Location**: .specify/specs/[feature-id]/spec.md  
**Status**: Draft | Ready for Plan  
**Input**: User description: "$ARGUMENTS"

> ROLE OF THIS FILE
> - Capture **WHAT** and **WHY** for exactly one bounded change (feature, bugfix, or refactor).  
> - Stay implementation-agnostic: no stacks, frameworks, file paths, or algorithms.  
> - Surface ambiguity explicitly with `[NEEDS CLARIFICATION: …]` instead of guessing.

---

## 1. Context & Intent

- **Problem / Motivation**: [What pain or opportunity this solves]
- **Target Users / Systems**: [Who or what benefits]
- **Interface Mode**: Vision-first UI | Contract-first API / Job / Batch
- **Scope (In)**:
  - [Bullets]
- **Scope (Out)**:
  - [Bullets]

> If UI exists, reference design assets (screenshots, mocks).  
> If backend-only, express intent via contracts and observable outcomes.

---

## 2. User Journeys (Slice-by-Slice)

> Each journey should be an independently testable slice.

### UJ-01 – [Title] (Priority: P1)

- **Narrative**: [Plain-language flow]
- **Value**: [Why this journey matters]
- **Independent Test**: [How to validate only this journey end-to-end]

**Acceptance Scenarios**

1. **Given** [initial state], **When** [action], **Then** [expected outcome]
2. **Given** […], **When** […], **Then** […]

---

### UJ-02 – [Title] (Priority: P2)

- **Narrative**: […]
- **Value**: […]
- **Independent Test**: […]

**Acceptance Scenarios**

1. **Given** […], **When** […], **Then** […]

---

[Add more journeys as needed: UJ-03, UJ-04, …]

---

## 3. Functional Requirements

> Describe observable behavior only. Do **not** pick technologies.

- **FR-001**: System MUST [behavior or capability]
- **FR-002**: System MUST [behavior or capability]
- **FR-003**: Users MUST be able to [key interaction]
- **FR-004**: System MUST [logging / audit / safety behavior]
- **FR-005**: System MUST [boundary behavior]
- **FR-006**: System MUST [NEEDS CLARIFICATION: specific question]

### Key Entities (If data is involved)

- **[Entity 1]**: [What this represents; key attributes in domain terms]
- **[Entity 2]**: [Relationships and invariants]

---

## 4. Non-Functional & Constraints

> Tech-agnostic but measurable targets.

- **NF-001 Performance**: [e.g., "UJ-01 completes < 2s in 95% of cases"]
- **NF-002 Reliability**: [e.g., "No data loss for events in scope"]
- **NF-003 Security / Privacy**: [e.g., "PII limited to …"]
- **NF-004 Compliance / Policy**: [e.g., "Audit trail retained for N days"]
- **NF-005 Observability**: [e.g., "Key events are traceable and measurable"]

---

## 5. Edge Cases & Failure Modes

- [Edge case / boundary input]
- [Error propagation behavior]
- [Timeout / degraded mode behavior]
- [Concurrency or race-condition scenarios]

---

## 6. Open Questions & Assumptions

> Use `[NEEDS CLARIFICATION: …]` for questions. Use assumptions sparingly.

- [NEEDS CLARIFICATION: …]
- [NEEDS CLARIFICATION: …]

**Assumptions**

- [Assumption 1 – why it is acceptable for now]
- [Assumption 2]

---

## 7. Readiness Checklist (Spec)

> Self-test before `/architect` runs.

- [ ] This spec describes exactly **one** bounded change for `[feature-id]`.
- [ ] All critical journeys (P1, P2, …) are listed with independent tests.
- [ ] Each FR/NF requirement is observable and testable (no vague language).
- [ ] Success for this feature is expressible via a small set of metrics.
- [ ] All known ambiguities are listed in **Open Questions**.
- [ ] `[NEEDS CLARIFICATION]` markers exist only where information truly does not exist.
- [ ] No implementation details (frameworks, DBs, file layouts) appear above.
- [ ] This spec is stable enough to generate `.specify/specs/[feature-id]/plan.md`.