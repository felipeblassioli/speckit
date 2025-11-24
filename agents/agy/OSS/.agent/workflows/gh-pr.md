---
description: "Generate PR content and open/update the Pull Request via GitHub MCP"
---

# Create/Update Pull Request

## 1. Generate Content (The Writer)
1.  **Read Artifacts**: `spec.md`, `plan.md`, `tasks.md`, `verify.md`.
2.  **Draft Body**: Create the PR description following `.specify/templates/PR.md`.
3.  **Draft Title**: `feat(<scope>): <short description from spec>`
4.  **Save Draft**: Write this to `.specify/specs/<feature-id>/PR.<repo>.md`.

## 2. Push Code (The Git Agent)
1.  **Check Status**: Ensure the working directory is clean or changes are committed.
2.  **Push**: Run `git push origin HEAD` (or specific branch).

## 3. Open PR (The GitHub Agent)
1.  **Check Existence**: Use `github.list_pull_requests` to see if a PR already exists for this branch.
2.  **Action**:
    * **If No PR**: Call `github.create_pull_request`:
        * `title`: [Draft Title]
        * `body`: [Content of PR.<repo>.md]
        * `head`: [Current Branch]
        * `base`: main (or user specified)
    * **If PR Exists**: Call `github.update_pull_request`:
        * Update the `body` with the latest content from `PR.<repo>.md`.

## 4. Final Output
* Display the link to the created/updated PR.