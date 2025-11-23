---
description: Create or update .specify/specs/<feature-id>/plan.md from an existing spec.md
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty). It may refine scope, priorities, or constraints for `<feature-id>`.

## Instructions

1. **Determine <feature-id>**
   - If the workflow explicitly provides `<feature-id>`, use it as-is.
   - Otherwise, infer it from context (current directory or spec path).

2. **Load context**
   - If `.specify/memory/constitution.md` exists, read it and respect its principles.
   - You MUST read `.specify/specs/<feature-id>/spec.md`. If it is missing, stop instead of guessing.

3. **Load the plan template**
   - Read `.specify/templates/plan.md` and treat its structure as **canonical**.

4. **Select the target file**
   - Target: `.specify/specs/<feature-id>/plan.md`.
   - If it exists, this run is an **update/refinement**.
   - If it does not exist, create it from `.specify/templates/plan.md`.

5. **Align with spec (WHAT/WHY → HOW)**
   - From `spec.md`, derive:
     - Context & Intent
     - User Journeys and priorities
     - Functional / Non-Functional requirements
     - Edge Cases & Failure Modes
   - Ensure every major element in `plan.md` traces back to something in `spec.md`.

6. **Populate or update the plan**
   - The plan MUST cover the same bounded change as `spec.md` for `<feature-id>`.
   - Fill all relevant sections from the template:
     - Design Summary
     - Technical Shape (Components, Data & Contracts, Execution Environment)
     - Story-by-Story Implementation Strategy
     - Foundation & Cross-Cutting Work
     - Testing & Verification Strategy
     - Risks, Trade-offs, TBDs
     - Plan Checklist
   - When updating, preserve useful existing architecture and rationale; rewrite only what is obsolete, conflicting, or unclear.

7. **Design at the correct abstraction level**
   - You MAY name components, services, modules by responsibility and define conceptual contracts.
   - You MUST NOT:
     - Paste large code samples or pseudo-code.
     - Over-specify library choices that belong in code.
     - Introduce unjustified abstractions that increase complexity.

8. **Make work parallelizable and explicit**
   - For each User Journey, create a small set of high-level tasks under "Story-by-Story Implementation Strategy".
   - Tasks MUST:
     - Be independently executable where possible.
     - Clearly state their scope (files/modules/areas) in a short label.
     - Use `[P]` in the text for tasks that can run in parallel.
   - Express dependencies in plain language (e.g., "depends on FND-01", "after P1-T02").

9. **Define foundations and cross-cutting concerns**
   - Under "Foundation & Cross-Cutting Work", list:
     - Blocking foundations (FND-xx).
     - Cross-cutting concerns (XCC-xx) such as error handling, observability, security/privacy.

10. **Define verification hooks**
    - Under "Testing & Verification Strategy", map each P1 journey to:
      - A Golden Path scenario.
      - Observable signals (responses, events, logs, metrics) that `/verify` can assert.
    - Add edge/failure scenarios for critical risks.

11. **Handle ambiguity and risks explicitly**
    - If the spec or user input leaves critical design aspects underspecified, add items to Risks/TBDs as:
      - `[NEEDS CLARIFICATION: <precise design question>]`
    - Call out intentional complexity or bigger architectural bets explicitly as trade-offs.

12. **Keep the structure clean**
    - Do NOT add arbitrary TODO lists outside the template's checkbox areas.
    - Do NOT create or modify any other file. This workflow only reads `.specify/specs/<feature-id>/spec.md`, `.specify/templates/plan.md`, `.specify/memory/constitution.md`, and writes `.specify/specs/<feature-id>/plan.md`.

13. **Validate and complete**
    - Use the Plan Checklist as a self-test.
    - Tick only items that are truly satisfied.
    - Ensure plan is readable in one sitting and ready for `/execute` and `/verify`.
    - Save `.specify/specs/<feature-id>/plan.md`.