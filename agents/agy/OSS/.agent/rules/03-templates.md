---
trigger: glob
globs: .specify/**/*
---

# Template Adherence

1.  **Canonical Structure**:
    * You MUST use the templates located in `.specify/templates/` as the absolute source of truth for file structure.
    * Do not delete sections. If a section is irrelevant, mark it `N/A`.

2.  **Checklists are Unit Tests**:
    * Every artifact (`spec`, `plan`, `verify`) contains a Readiness Checklist.
    * You MUST NOT mark a checkbox `[x]` unless you have rigorously verified that condition.
    * Faking a checklist is a violation of the Core Constitution.

3.  **Feature Identity**:
    * All work must occur within `.specify/specs/<feature-id>/`.
    * Do not create files in the root `.specify/` folder.