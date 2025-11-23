---
description: Execute verification steps from plan.md and produce a strict audit report in verify.md
---

## User Input

```text
$ARGUMENTS
```

## Instructions

1.  **Setup & Context**

      - Identify `<feature-id>` from the directory or arguments.
      - Read `.specify/specs/<feature-id>/spec.md` (for intent) and `plan.md` (for Golden Paths/Failure Scenarios).
      - Read `tasks.md` to confirm implementation is complete (`- [x]`).

2.  **Select Target File**

      - Target: `.specify/specs/<feature-id>/verify.md`.
      - If it exists, append a new "Run" section. If not, create it using the **Strict Template** below.

3.  **Execute Verification**

      - Identify P1/Golden Paths (GP-xx) and Failure Scenarios (EF-xx) from `plan.md`.
      - **Action**: Simulate or execute the checks (depending on arguments).
      - **Observe**: Gather specific evidence (logs, exit codes, screenshots).

4.  **Write Report (STRICT FORMAT)**

      - You **MUST** use the Markdown structure below.
      - You **MUST** fill the "Run Context" (Who, Date, Env).
      - You **MUST** include the "Results Summary" table.
      - **Do not** summarize lazily; list every path checked.

-----

## Strict Template (Do Not Deviate)

# Verify: [Feature Name]

**Feature ID**: [feature-id]
**Spec**: spec.md
**Plan**: plan.md

## 1\. Run Context

  - **Date**: [YYYY-MM-DD HH:MM]
  - **Agent/User**: [Name]
  - **Environment**: [Local/Staging/Prod]
  - **Scope**: [Full | Smoke | Specific Journey]

## 2\. Results Summary

| ID | Type | Status | Notes |
| :--- | :--- | :--- | :--- |
| GP-01 | Golden | ✅ Pass / ❌ Fail | [One line summary] |
| EF-01 | Failure | ✅ Pass / ❌ Fail | [One line summary] |

## 3\. Detailed Results

### [ID] – [Short Title]

  - **Expected**: [From plan.md]
  - **Observed**: [What actually happened]
  - **Status**: ✅ / ❌
  - **Evidence**:
    ```text
    [Paste log snippets, exit codes, or curl responses here]
    ```

## 4\. Risks & Regressions

  - [ ] No regressions observed in related features.
  - [ ] [Issue Description if any]

<!-- end list -->