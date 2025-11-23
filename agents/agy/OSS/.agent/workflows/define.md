---
description: Create or update .specify/specs/<feature-id>/spec.md from user intent
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty). It describes a single bounded change (feature, bugfix, or refactor) and MAY provide `<feature-id>`.

## Instructions

1. **Determine <feature-id>**
   - If the workflow explicitly provides `<feature-id>`, use it as-is.
   - Otherwise, infer a short, stable slug from the request (e.g., `drone-exec-debug-trace`).

2. **Load context**
   - If `.specify/memory/constitution.md` exists, read it and respect its constraints.

3. **Load the spec template**
   - Read `.specify/templates/spec.md` and treat its structure as **canonical**.

4. **Select the target file**
   - Target: `.specify/specs/<feature-id>/spec.md`.
   - If it exists, this run is an **update/refinement**.
   - If it does not exist, create it from `.specify/templates/spec.md`.

5. **Populate or update the spec**
   - The spec MUST describe **exactly one** bounded change for `<feature-id>`.
   - Fill all relevant sections from the template:
     - Context & Intent
     - User Journeys
     - Functional Requirements
     - Non-Functional & Constraints
     - Edge Cases & Failure Modes
     - Open Questions & Assumptions
     - Readiness Checklist
   - When updating, preserve useful existing content; rewrite only what is obsolete, conflicting, or incomplete.

6. **Respect abstraction level (WHAT/WHY only)**
   - Focus on behavior and outcomes, not implementation.
   - You MUST NOT introduce:
     - Specific technologies, frameworks, or databases.
     - File/directory layouts or module names.
     - Low-level API signatures or algorithms.

7. **Handle ambiguity explicitly**
   - Whenever critical information is missing, insert:
     - `[NEEDS CLARIFICATION: <precise question>]`
   - Do NOT guess or silently assume behavior.

8. **Make the spec testable and measurable**
   - For each User Journey, add at least one Given/When/Then acceptance scenario.
   - Phrase Functional Requirements in terms of observable behavior.
   - Make Non-Functional requirements measurable where possible.

9. **Keep the structure clean**
   - Do NOT add implementation steps, task checklists, or `- [ ]` TODOs outside the Readiness Checklist.
   - Do NOT touch any other file. This workflow only reads `.specify/templates/spec.md` and `.specify/memory/constitution.md`, and writes `.specify/specs/<feature-id>/spec.md`.

10. **Validate and complete**
    - Use the Readiness Checklist in the spec as a self-test.
    - Tick only items that are truly satisfied.
    - If blocking `[NEEDS CLARIFICATION]` remain, keep `Status: Draft`.
    - Save `.specify/specs/<feature-id>/spec.md`.