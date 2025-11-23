# Plan: [SHORT NAME]

**Feature ID**: [feature-id]
**Location**: .specify/specs/[feature-id]/plan.md  
**Spec**: ../spec.md  
**Status**: Draft | Ready for Execute

> ROLE OF THIS FILE
> - Translate spec **WHAT/WHY** into a concrete technical approach.  
> - Provide enough structure for `/execute` and `/verify` to coordinate agents.  
> - Stay high-level; push large code/config into separate artifacts.

---

## 1. Design Summary

- **Problem Restatement**: [1–3 sentences derived from spec]
- **Overall Approach**: [High-level architecture]
- **Interface Shape**: [UI modules, APIs, jobs, events]
- **Key Dependencies / External Systems**: [Only where they affect contracts or constraints]

---

## 2. Technical Shape

### 2.1 Components

> List main components at a level where separate agents can own them.

- **CMP-01 [Name]**: [Responsibility and inputs/outputs]
- **CMP-02 [Name]**: [Responsibility and inputs/outputs]
- **CMP-03 [Name]**: [Responsibility and inputs/outputs]

### 2.2 Data & Contracts

- **Domain Entities**: [Short recap aligned with spec entities]
- **Core Contracts** (conceptual; details live in code/contract artifacts):
  - **CTR-01**: [API / event / message: purpose + caller]
  - **CTR-02**: […]

### 2.3 Execution Environment (Minimal)

- **Runtime**: [e.g., "Server-side service", "Browser SPA", "Batch job"]
- **Deployment Surface**: [e.g., "Containerized HTTP service", "Background worker", "CLI tool"]
- **Notable Constraints**: [Latency, cost, platform limits]

---

## 3. Story-by-Story Implementation Strategy

> Map journeys from `../spec.md` to concrete work.

### UJ-01 – [Title] (Priority: P1)

- **Goal**: [What success looks like]
- **Dependencies**: [Foundations, contracts, components required]

**Implementation Steps (High-Level Tasks)**

- [ ] P1-T01 [P] [Scope] – [Short description]
- [ ] P1-T02 [P] [Scope] – [Short description]
- [ ] P1-T03 [Depends on P1-T01, P1-T02] – [Short description]

**Verification Hooks**

- [ ] `/verify` can run [scenario] via [UI/contract].
- [ ] Metrics/logs exist to detect failure for this journey.

---

### UJ-02 – [Title] (Priority: P2)

- **Goal**: […]
- **Dependencies**: […]

**Implementation Steps (High-Level Tasks)**

- [ ] P2-T01 [P] [Scope] – [Short description]
- [ ] P2-T02 [Scope] – [Short description]

**Verification Hooks**

- [ ] `/verify` scenario: […].

---

[Repeat for UJ-03, UJ-04, …]

---

## 4. Foundation & Cross-Cutting Work

### 4.1 Foundations (Blocking)

- [ ] FND-01 – [e.g., Minimal persistence/state handling]
- [ ] FND-02 – [e.g., Authentication/identity]
- [ ] FND-03 – [e.g., Basic telemetry: logs, traces, metrics]

### 4.2 Cross-Cutting Concerns

- [ ] XCC-01 – Error handling strategy
- [ ] XCC-02 – Observability (logging/tracing/metrics)
- [ ] XCC-03 – Security/privacy considerations

---

## 5. Testing & Verification Strategy

### 5.1 Golden Paths

- **GP-01**: [Maps to UJ-01; how to execute and what to observe]
- **GP-02**: [Maps to UJ-02]

### 5.2 Edge / Failure Scenarios

- **EF-01**: [Failure condition + expected system behavior]
- **EF-02**: […]

### 5.3 Automation Surface

- [ ] What can be exercised via browser automation (if UI exists)
- [ ] What can be exercised via contract-level calls (APIs, events, CLI)

---

## 6. Risks, Trade-offs, and TBDs

- **Risk-01**: [Description] – [Mitigation or monitoring]
- **Risk-02**: […]

**TBDs / Follow-Ups**

- [NEEDS CLARIFICATION: …]
- [NEEDS CLARIFICATION: …]

---

## 7. Plan Checklist (Pre-/execute Gate)

### 7.1 Alignment & Simplicity

- [ ] Every item in this plan traces back to `../spec.md` for `[feature-id]`.
- [ ] No speculative "maybe later" features are planned here.
- [ ] The number of components is minimal for the problem.
- [ ] Any intentional complexity is listed in **Risks / Trade-offs**.

### 7.2 Clarity for Agents

- [ ] Each high-level task clearly names its scope (files/modules/areas).
- [ ] Parallelizable tasks are marked with `[P]`.
- [ ] Foundations that must precede other work are under **4.1 Foundations**.
- [ ] `/verify` hooks exist for each P1 journey.

### 7.3 Readiness

- [ ] All critical `[NEEDS CLARIFICATION]` items that block implementation are resolved.
- [ ] This plan can be executed by independent agents without hidden context.
- [ ] This plan is small enough to be readable in one sitting.