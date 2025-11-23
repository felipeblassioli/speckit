---
description: Execute tasks for .specify/specs/<feature-id>/ based on tasks.md, guided by plan.md Golden Paths
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty). It may constrain scope (phase, user journey, files) or execution mode (build-only vs. include verification hooks).

## Instructions

1. **Determine <feature-id> and scope**
   - If the workflow explicitly provides `<feature-id>`, use it.  
   - Otherwise, infer from current directory or explicit path in `$ARGUMENTS`.  
   - Parse `$ARGUMENTS` for optional scope hints:
     - Target phase(s): e.g., "Phase 3", "User Journey 1".  
     - Target user story: e.g., `US1`, `US2`.  
     - Mode: e.g., "build-only", "build-and-verify".

2. **Load context**
   - Read `.specify/memory/constitution.md` if present and respect its rules (simplicity, anti-abstraction, etc.).  
   - Read `.specify/specs/<feature-id>/spec.md` to understand journeys and priorities.  
   - Read `.specify/specs/<feature-id>/plan.md` to get:
     - Components and contracts.  
     - Golden Paths and edge/failure scenarios from **Testing & Verification Strategy**.  
   - Read `.specify/specs/<feature-id>/tasks.md` as the **single source of tasks**.

3. **Parse tasks.md**
   - Identify all tasks lines matching the required format:  
     `- [ ] T### [P?] [USn?] Description`.  
   - Classify tasks by:
     - Phase (Setup, Foundations, UJ phases, Polish) based on headings.  
     - Story label (`US1`, `US2`, …) if present.  
     - Parallelization marker `[P]`.
   - Determine which tasks are:
     - **Pending** – unchecked.  
     - **Completed** – checked (`- [x]`).

4. **Respect phase ordering and dependencies**
   - You MUST NOT execute UJ tasks before all blocking Foundations in Phase 2 are complete.  
   - Within a phase, prefer executing tasks in ID order unless `$ARGUMENTS` or explicit text indicates otherwise.  
   - Only schedule parallel tasks when their file/scope clearly does not conflict.

5. **Prioritize Golden Paths and verification hooks**
   - From `../plan.md`, find P1 journeys and their Golden Paths (GP-xx) and failure scenarios (EF-xx).  
   - In `tasks.md`, locate tasks that implement or exercise these paths (e.g., tasks under "Verification Tasks" sections).  
   - When choosing work for `/execute`, prioritize sequences that:
     - Complete all implementation tasks necessary for at least one Golden Path.  
     - Then complete the associated verification tasks.

6. **Execute tasks in small, coherent batches**
   - Work in small batches of tasks that share a phase and, ideally, a single user story.  
   - For each batch:
     1. Re-read plan.md sections relevant to those tasks (components/contracts).  
     2. Apply changes to code/docs/tests exactly as described.  
     3. Keep changes minimal and scoped to the task description.
   - After completing a task, mark it as done by changing `- [ ]` → `- [x]` in `.specify/specs/<feature-id>/tasks.md`.

7. **Maintain alignment with spec and plan**
   - If you encounter a task that is clearly inconsistent with `spec.md` or `plan.md`:
     - Do NOT silently implement it.  
     - Instead, adjust the task text to align with the spec/plan **or** add a short note in the description pointing to the ambiguity.  
   - If a critical gap is discovered (task refers to non-existent journey or component), you MAY add a new task at the appropriate phase to close the gap, but keep this rare and grounded in the plan.

8. **Verification mode (when requested)**
   - If `$ARGUMENTS` indicates "build-and-verify" or similar:
     - For each completed journey (all its tasks checked), trigger the relevant Golden Path(s) as described in `../plan.md`.  
     - Ensure that verification-related tasks in `tasks.md` are also executed and checked.  
     - Record any deviations or failures as inline comments in tasks.md or in the appropriate docs/tests.

9. **Keep the structure clean**
   - Do NOT change the heading structure of `.specify/specs/<feature-id>/tasks.md`.  
   - Do NOT change the task format described in section **0. Task Format**.  
   - Do NOT create or modify `.specify/specs/<feature-id>/spec.md` or `plan.md` from this workflow. Only read them.

10. **Completion for a run**
    - A single `/execute` invocation does **not** need to finish the entire feature. It should:
      - Complete a coherent set of tasks (often a subset of a phase or a full journey).  
      - Update checkboxes in `tasks.md` accordingly.  
      - Keep the file consistent and readable for the next `/execute` or `/verify` run.
    - At natural boundaries (end of a journey or phase), ensure:
      - All related tasks are either checked or explicitly deferred.  
      - Golden Path tasks for that journey are implemented if requested.

11. **Safety & Simplicity guardrails**
    - Prefer the simplest implementation that satisfies the spec and plan.  
    - Do not introduce new frameworks or major architectural shifts unless explicitly called for in `plan.md`.  
    - If you believe the plan is too complex or misaligned with the constitution, stop and surface that concern instead of silently implementing it.