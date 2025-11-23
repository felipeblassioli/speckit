# Tasks: [SHORT NAME]

**Feature ID**: [feature-id]  
**Location**: .specify/specs/[feature-id]/tasks.md  
**Spec**: ../spec.md  
**Plan**: ../plan.md  
**Status**: Draft | In Progress | Complete

> ROLE OF THIS FILE
> - This is the **canonical work graph** for `/execute` agents.  
> - Tasks here are the only source of truth for what should be built and in what order.  
> - Structure and formatting are strict so agents can safely parallelize work.

---

## 0. Task Format

Every task MUST follow this format exactly:

```text
- [ ] T### [P?] [USn?] Description with file path or scope
```

- `- [ ]` – Markdown checkbox (always present).  
- `T###` – Zero-padded task ID, e.g., `T001`, `T002`. IDs are unique within this file.  
- `[P]` – Optional marker: task can be run in parallel (no dependency on unfinished tasks; different files/scope).  
- `[USn]` – Optional user-story label (e.g., `[US1]`, `[US2]`) for story phases only.  
- **Description** – Clear action that names file(s) or scope, e.g. `src/services/user_service.ts` or `frontend/src/pages/LoginPage.tsx`.

Examples:

- `- [ ] T001 Create project structure as defined in ../plan.md`
- `- [ ] T005 [P] Configure CI workflow in .github/workflows/ci.yaml`
- `- [ ] T012 [P] [US1] Implement LoginForm component in frontend/src/components/LoginForm.tsx`
- `- [ ] T017 [US1] Wire login endpoint in backend/src/routes/auth.ts (depends on T015)`

---

## 1. Phase 1 – Setup (Shared Infrastructure)

**Purpose**: Repository- or project-level initialization needed before any real feature work.

- [ ] T001 [P] Initialize project/tooling per ../plan.md
- [ ] T002 [P] Add baseline linting/formatting configuration in [path]
- [ ] T003 [P] Setup basic CI checks in [path]

---

## 2. Phase 2 – Foundations (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be in place before user journeys can be implemented.

> No user story work should start until this phase is complete.

- [ ] T010 Define core domain types/entities in [path]
- [ ] T011 [P] Establish persistence or state layer in [path]
- [ ] T012 [P] Implement minimal authentication/identity handling in [path]
- [ ] T013 [P] Setup basic observability (logging/tracing/metrics) in [path]

---

## 3. Phase 3 – User Journey 1 – [UJ-01 Title] (Priority: P1)

**Goal**: [What this journey delivers when complete]  
**Independent Test**: [How to verify this journey end-to-end]

### 3.1 Verification Tasks (Golden Path / Edge Hooks)

> Derive from `Testing & Verification Strategy` in ../plan.md (e.g., GP-01, EF-01).

- [ ] T020 [P] [US1] Implement GP-01 verification path in [test or script path]
- [ ] T021 [P] [US1] Implement EF-01/EF-02 failure-path checks in [test or script path]

### 3.2 Implementation Tasks

- [ ] T022 [P] [US1] Implement core domain logic for UJ-01 in [path]
- [ ] T023 [P] [US1] Add interface / endpoint / UI surface for UJ-01 in [path]
- [ ] T024 [US1] Integrate all UJ-01 pieces and ensure telemetry for Golden Path in [path]

**Checkpoint**: UJ-01 should now be independently executable and observable.

---

## 4. Phase 4 – User Journey 2 – [UJ-02 Title] (Priority: P2)

**Goal**: [What this journey delivers]  
**Independent Test**: [How to verify this journey end-to-end]

### 4.1 Verification Tasks

- [ ] T030 [P] [US2] Implement GP-02 verification path in [path]

### 4.2 Implementation Tasks

- [ ] T031 [P] [US2] Implement domain logic for UJ-02 in [path]
- [ ] T032 [US2] Add interface / endpoint / UI surface for UJ-02 in [path]

**Checkpoint**: UJ-02 should now be independently executable and observable.

---

[Repeat User Journey phases as needed: US3, US4, …]

---

## 5. Phase N – Polish & Cross-Cutting Concerns

**Purpose**: Work that spans multiple journeys and does not belong to a single story.

- [ ] T090 [P] Documentation updates in docs/[feature-id]/
- [ ] T091 Code cleanup/refactors in [paths]
- [ ] T092 [P] Performance tuning across key paths in [paths]
- [ ] T093 [P] Additional safety/guardrails (rate limiting, validation) in [paths]

---

## 6. Dependencies & Parallelization Summary

> Keep this section short but explicit. `/execute` agents use it to schedule work.

- **Phase ordering**: 1 (Setup) → 2 (Foundations) → 3+ (User Journeys) → N (Polish).  
- **Blocking rule**: Phase 2 must be completed before any UJ phase starts.  
- **Parallelization**:
  - Tasks marked `[P]` may run in parallel when their file scopes do not conflict.  
  - Different User Journeys may be worked in parallel once Foundations are complete.  

---

## 7. Tasks Checklist (Quality Gate)

- [ ] All tasks follow the exact format: `- [ ] T### [P?] [USn?] Description with file path or scope`.
- [ ] Every task is small enough for a single agent to complete in isolation.
- [ ] Every P1 User Journey in ../spec.md has at least one verification task linked to a Golden Path.
- [ ] Dependencies between tasks are either implicit in phase ordering or clearly stated in descriptions.
- [ ] No vague tasks like "implement feature" without file/scope.
- [ ] This file alone is sufficient for `/execute` to orchestrate work for `[feature-id]`.