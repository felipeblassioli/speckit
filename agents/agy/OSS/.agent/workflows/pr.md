---
description: Generate a Pull Request description for .specify/specs/<feature-id> in the current repo, using spec.md, plan.md, tasks.md, and verify.md, and keep it consistent across multi-repo features
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty). It may specify:
- `<feature-id>` to target (required for multi-repo features).  
- The role of this repo in the feature (e.g., `primary=cli`, `dependency=engine`).  
- Hints for PR title style (e.g., Conventional Commit-like vs free-form).

## Instructions

1. **Determine <feature-id> and repository context**
   - If `<feature-id>` is provided in `$ARGUMENTS`, use it.  
   - Otherwise, infer from the `.specify/specs/<feature-id>/` directory in the current repo. If multiple candidates exist, stop and ask for clarification.  
   - Determine the repo name using:
     - `git config --get remote.origin.url` (for human-readable name), and/or  
     - The top-level directory name as a fallback.

2. **Load core artifacts**
   - Read `.specify/specs/<feature-id>/spec.md` to extract:
     - Problem / Motivation  
     - User Journeys (UJ-xx) and their priorities  
     - Key Functional Requirements (FR-xx)
   - Read `.specify/specs/<feature-id>/plan.md` to extract:
     - Components (CMP-xx) relevant to this repo  
     - Testing & Verification Strategy (Golden Paths GP-xx, Failure Scenarios EF-xx)
   - Read `.specify/specs/<feature-id>/tasks.md` to extract:
     - Tasks completed in this repo (by ID and description)  
     - Any tasks that appear out of scope for this repo (ignore them for PR body)
   - Read `.specify/specs/<feature-id>/verify.md` if it exists to extract:
     - Which Golden Paths and failure scenarios were executed and their status  
     - Any regressions, side effects, or open issues.

3. **Determine this repo's slice of the feature**
   - From `plan.md` components and `tasks.md` descriptions, identify which parts of the global feature this repo owns, e.g.:
     - CLI flags, help text, UX changes.  
     - Engine or library behavior, logging, or contracts.  
   - Summarize this as **Scope (This Repo)** in one or two sentences.

4. **Inspect the working tree (optional but recommended)**
   - Run read-only git commands to enrich the PR body:
     - `git status --porcelain=v1`  
     - `git diff --name-only origin/<base-branch>...HEAD` (or equivalent)  
   - Use file paths and directory names to:
     - Confirm which components/files listed in `plan.md` and `tasks.md` were actually touched.  
     - Infer scopes (e.g., `cmd/drone/exec`, `engine/engine.go`).

5. **Choose a PR title pattern**
   - Prefer a concise, user-facing title of the form:
     - `[feature-id] – <short description>`  
     - or a Conventional Commit-inspired style: `<type>(<scope>): <subject>` (if `$ARGUMENTS` requests this).
   - The title should:
     - Reflect the user-facing intent (from `spec.md`), not implementation details.  
     - Be reusable across repos by varying the scope (e.g., `exec`, `engine`).

6. **Populate the PR template**
   - Load `.specify/templates/PR.md` and treat its structure as canonical.  
   - Fill the following sections using the artifacts:

   **Header**
   - `[feature-id]` from directory or `$ARGUMENTS`.  
   - `[repo-name]` from git remote or folder name.  
   - Set Spec/Plan/Tasks/Verify paths as shown in the template.

   **1. Summary**
   - Derive **Problem** and **Goal** from `spec.md` Context & Intent.  
   - Describe **Scope (This Repo)** based on plan components and tasks.  
   - Ensure the wording is consistent with other repos by reusing phrasing from `spec.md` wherever possible.

   **2. User-Facing Changes**
   - List relevant User Journeys from `spec.md` that this repo contributes to.  
   - For a primary repo (e.g., CLI), include concrete flags/endpoints/UI names.  
   - For dependency repos (e.g., engine), focus on how behavior supports the primary UX (e.g., enhanced logging, new hooks).

   **3. Internal Changes (This Repo)**
   - From `plan.md` components and `tasks.md`, list:
     - Components (CMP-xx) that map to this repo.  
     - Key files/modules touched (from git diff + task descriptions).  
   - Summarize completed tasks as bullet points with their IDs.

   **4. Verification**
   - Use `verify.md` to list Golden Paths (GP-xx) and failure scenarios (EF-xx) executed in this repo's context.  
   - Mark each as ✅/❌ and reference evidence (
     logs, tests, screenshots, trace IDs) when available.

   **5. Compatibility, Versioning & Migration**
   - If a version bump is present (e.g., in `go.mod`, `package.json`):
     - Extract before/after versions and include them.  
   - State whether the change is backward compatible for existing callers.  
   - If there is a breaking change, describe the migration steps.

   **6. Risks & Follow-Ups**
   - Pull known risks from `plan.md` and open issues/regressions from `verify.md`.  
   - Only include items relevant to this repo; cross-repo concerns can be summarized with links to other PRs.

   **7. Checklist**
   - Ensure the checklist items reflect the state of artifacts in this repo.  
   - Do **not** mark items as complete if the corresponding artifact is missing or obviously stale.

7. **Multi-repo consistency

   - For a multi-repo feature (e.g., CLI + engine):
     - Ensure all repos share the same `feature-id` and refer back to the same `spec.md` intent.  
     - Use consistent language for **Problem** and **Goal** across PRs.  
     - Differentiate PRs by their **Scope (This Repo)** and component list.
   - If `$ARGUMENTS` provides a list of related repos or PR URLs, include them in the **Related Repos** field of the template.

8. **Output

   - Produce a finalized PR description in Markdown following `.specify/templates/PR.md`.  
   - Do **not** execute any git push or PR creation commands; your role is to generate the content only.  
   - If requested, you MAY also write the generated PR body to a local file such as:
     - `.specify/specs/<feature-id>/PR.<repo-name>.md`
   - Clearly label which part is the **PR title** and which is the **PR body** so the user can paste them into their Git host.

9. **Safety & Non-Goals

   - Do NOT invent new behavior that is not reflected in `spec.md` or `plan.md`.  
   - Do NOT claim tests or verification that are not present in `verify.md`.  
   - Do NOT modify `spec.md`, `plan.md`, `tasks.md`, or `verify.md`; they are read-only for this workflow.