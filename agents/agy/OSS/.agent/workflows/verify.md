---
description: Verification report template for a feature, driven by spec.md, plan.md, and tasks.md
---

# Verify: [SHORT NAME]

**Feature ID**: [feature-id]  
**Location**: .specify/specs/[feature-id]/verify.md  
**Spec**: ../spec.md  
**Plan**: ../plan.md  
**Tasks**: ../tasks.md  
**Environment**: [local | staging | production | other]  
**Run ID**: [yyyy-mm-ddThh:mm or unique label]

> ROLE OF THIS FILE
> - Summarize **what was actually exercised** for this feature.  
> - Connect Golden Paths and failure scenarios from plan.md to concrete checks.  
> - Provide a concise, auditable record for `/verify` runs.

---

## 1. Run Context

- **Who / Agent**: [human or agent identifier]
- **Date / Time**: [timestamp]
- **Scope**: [Full regression | P1 only | Single journey (UJ-0x) | Smoke]
- **Code Revision**: [commit SHA or tag]
- **Relevant Config**: [branch, feature flags, notable env settings]

---

## 2. Journeys & Golden Paths in Scope

> Derived from `../plan.md` (Testing & Verification Strategy) and implementation status in `../tasks.md`.

- **UJ-01 [Title]**
  - Golden Paths: [GP-01, GP-0x]
  - Failure Scenarios: [EF-01, EF-0x]
- **UJ-02 [Title]**
  - Golden Paths: [GP-02, GP-0y]
- [...]

---

## 3. Results Summary

> High-level view of which paths passed or failed.

| Journey | Path ID | Type      | Status | Notes |
| :------ | :------ | :-------- | :----- | :---- |
| UJ-01   | GP-01   | Golden    | ✅/❌   | [Short note] |
| UJ-01   | EF-01   | Failure   | ✅/❌   | [Short note] |
| UJ-02   | GP-02   | Golden    | ✅/❌   | [Short note] |

---

## 4. Detailed Results by Path

### 4.1 UJ-01 – [Title]

#### GP-01 – [Short Name]

- **Intent** (from plan.md): [What this path is supposed to prove]
- **Steps Executed**:  
  1. [Step 1]
  2. [Step 2]
  3. [Step 3]
- **Observed Outcome**: [What actually happened]
- **Status**: ✅ Pass | ❌ Fail
- **Evidence**: [links to logs, screenshots, traces, test runs]

#### EF-01 – [Failure Scenario]

- **Intent**: [What failure behavior is expected]
- **Steps Executed**:  
  1. [Step 1]
  2. [Step 2]
- **Observed Outcome**: [What actually happened]
- **Status**: ✅/❌
- **Evidence**: [...]

---

### 4.2 UJ-02 – [Title]

[Repeat structure above]

---

## 5. Observability Check

> Do we see the signals we expect in logs, traces, metrics?

- **Logging**: [Are key events clearly logged? Any gaps?]
- **Tracing**: [Are traces/span attributes sufficient to follow the Golden Paths?]
- **Metrics**: [Are there counters/latency/error-rate metrics tied to journeys?]
- **Dashboards/Alerts**: [Any existing dashboards/alerts that reflect this feature?]

---

## 6. Regressions & Side Effects

- [Any existing behavior broken or degraded by this feature?]
- [Interactions with other journeys or features.]

---

## 7. Open Issues & Follow-Ups

- [NEEDS CLARIFICATION: …]
- [BUG: link or description]
- [TECH DEBT: note]

---

## 8. Verification Checklist

- [ ] All in-scope Golden Paths (GP-xx) from `../plan.md` were executed.  
- [ ] All in-scope failure scenarios (EF-xx) were executed or explicitly marked as out-of-scope.  
- [ ] Results for each path are recorded with clear pass/fail status.  
- [ ] At least one P1 journey has a fully passing Golden Path.  
- [ ] Observability is sufficient to debug failures along these paths.  
- [ ] Any regressions or open issues are captured above.  
- [ ] This report is self-contained and can be read without additional chat logs.
```

---

# .specify/verify.md

```md
---
description: Verify .specify/specs/<feature-id> against spec.md and plan.md using Golden Paths and tasks.md, and write/update verify.md.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty). It may specify `<feature-id>`, scope (P1 only, specific journeys), environment, or mode (dry-run vs actual verification).

## Instructions

1. **Determine <feature-id> and scope**
   - If `<feature-id>` is provided by the workflow or `$ARGUMENTS`, use it.  
   - Otherwise, infer it from the current directory or explicit file paths.  
   - From `$ARGUMENTS`, extract optional scope:
     - Journeys (e.g., `UJ-01`, `UJ-02`).  
     - Priority (e.g., `P1-only`).  
     - Environment (local/staging/prod).  
     - Mode (e.g., `smoke`, `full`, `dry-run`).

2. **Load context**
   - Read `.specify/memory/constitution.md` if present and respect its principles.  
   - Read `.specify/specs/<feature-id>/spec.md` to understand journeys and expected behavior.  
   - Read `.specify/specs/<feature-id>/plan.md` to:
     - Identify Golden Paths (GP-xx) and failure scenarios (EF-xx) from **Testing & Verification Strategy**.  
     - Determine which journeys and components are P1.
   - Read `.specify/specs/<feature-id>/tasks.md` to:
     - Confirm that implementation and verification tasks for a given path are completed (`- [x]`).  
     - Avoid treating incomplete paths as fully verifiable.

3. **Load or create verify.md**
   - Target file: `.specify/specs/<feature-id>/verify.md`.  
   - If it exists, update it for the current run (append a new Run ID section or update the latest).  
   - If it does not exist, create it from `.specify/templates/verify.md` and fill header fields (Feature ID, Environment, Run ID).

4. **Select Golden Paths and scenarios in scope**
   - From plan.md, build a list of candidate paths:
     - All P1 journeys’ Golden Paths (GP-xx).  
     - Failure scenarios (EF-xx) where explicitly requested or critical.  
   - Filter this list according to `$ARGUMENTS` scope (e.g., only UJ-01, or P1-only).  
   - For each candidate path, check tasks.md to ensure corresponding verification tasks exist and are marked complete. If they are not complete:
     - Mark the path as **blocked** in the report with a clear note.

5. **Execute in verification mode**
   - For each **unblocked** path in scope:
     - Follow the steps described in plan.md’s Testing & Verification Strategy for that path.  
     - Use the available automation surface:
       - If the path is UI-based, use the browser/automation specified in plan.md.  
       - If contract-based (API/CLI/event), invoke the described interfaces and payloads.  
     - Observe actual outcomes: responses, side effects, logs, traces, metrics.
   - For `dry-run` mode: instead of executing, simulate steps and record what **would** be run and which signals would be checked.

6. **Record results into verify.md**
   - Update **Run Context** with:
     - Date/Time, Environment, Scope, Code Revision (if known).  
   - In **Journeys & Golden Paths in Scope**, list all paths considered and whether they were run or blocked.  
   - In **Results Summary**, add or update rows per path:
     - Journey, Path ID, Type (Golden/Failure), Status (✅/❌/⏳ for blocked), Short note.  
   - In **Detailed Results**, for each executed path:
     - Capture steps executed, observed outcome, explicit Pass/Fail, and evidence references (logs, screenshots, test output, trace IDs).

7. **Check observability and regressions**
   - Use logs/traces/metrics described in plan.md and any feature-specific dashboards to confirm:
     - Signals exist for the executed paths.  
     - There are no obvious regressions or unexpected errors linked to this feature.  
   - Summarize observations in the **Observability Check** and **Regressions & Side Effects** sections.

8. **Handle ambiguity and gaps**
   - If plan.md describes a Golden Path but there is no clear way to execute it (missing endpoint, unclear steps):
     - Do NOT invent a new behavior.  
     - Mark it as blocked and add a `[NEEDS CLARIFICATION: …]` entry under **Open Issues & Follow-Ups** in verify.md.  
   - If verification reveals behavior that contradicts spec.md:  
     - Record the discrepancy clearly as a BUG or Open Issue, referencing the relevant FR/NF or journey.

9. **Respect file boundaries**
   - `/verify` MUST NOT modify `.specify/specs/<feature-id>/spec.md`, `plan.md`, or `tasks.md`.  
   - It only reads those files and writes updates to `.specify/specs/<feature-id>/verify.md`.

10. **Verification Checklist and completion**
    - Use the **Verification Checklist** in verify.md as a self-test.  
    - Tick only items that are actually satisfied by this run.  
    - A single `/verify` invocation does **not** need to cover all paths, but it MUST leave verify.md in a coherent state, clearly indicating:
      - Which paths were executed and their results.  
      - Which paths are blocked and why.  
      - Any new open issues or regressions discovered.
    - Save `.specify/specs/<feature-id>/verify.md` with the updated content.