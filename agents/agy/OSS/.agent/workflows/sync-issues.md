---
description: "Sync tasks.md to GitHub Issues using the GitHub MCP"
trigger: "/sync-issues"
tools: ["github"]
---

# Sync Tasks to Issues

## Context
* **Feature ID**: [Extract from current directory]
* **Task File**: `.specify/specs/<feature-id>/tasks.md`

## Procedure

1.  **Parse `tasks.md`**
    * Identify all tasks (lines starting with `- [ ] [ID]`).
    * Check if they already have an Issue Reference (e.g., `(Issue #123)`).

2.  **Check Existing Issues (Idempotency)**
    * Use `github.search_issues` to find open issues for this `<feature-id>`.
    * Map existing issues to your task list to avoid duplicates.

3.  **Create Missing Issues**
    * For each task *without* an issue link:
        * **Title**: `[<feature-id>] <Task ID>: <Short Description>`
        * **Body**:
            ```markdown
            **Spec**: spec.md
            **Plan**: plan.md
            **Original Task**: [Task Description]

            This issue tracks a specific task for feature `<feature-id>`.
            See `.specify/specs/<feature-id>/` for full context.
            ```
        * **Labels**: `specify`, `<feature-id>`
        * **Action**: Call `github.create_issue`.

4.  **Update `tasks.md`**
    * Rewrite the line in `tasks.md` to append the new Issue URL/Number.
    * **Example**: `- [ ] P1-T01 Implement CLI flags` -> `- [ ] P1-T01 Implement CLI flags (#45)`

5.  **Report**
    * List the issues created or linked.