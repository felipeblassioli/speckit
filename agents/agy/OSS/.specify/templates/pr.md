---
description: "Pull Request description template for a feature, driven by Specify/Antigravity artifacts (spec, plan, tasks, verify)."
---
# [feature-id] – [Short, user-facing PR title]

**Feature ID**: [feature-id]  
**Repo**: [repo-name]  
**Spec**: .specify/specs/[feature-id]/spec.md  
**Plan**: .specify/specs/[feature-id]/plan.md  
**Tasks**: .specify/specs/[feature-id]/tasks.md  
**Verify**: .specify/specs/[feature-id]/verify.md  
**Related Repos** (if multi-repo feature): [e.g., drone-cli, drone-runner-docker]

---

## 1. Summary

> One or two paragraphs, derived from spec.md (Context & Intent + Journeys).

- **Problem**: [What was missing or painful]
- **Goal**: [What this feature enables for users]
- **Scope (This Repo)**: [What slice of the global feature this repo implements]

_Example (for reference):_  
"Expose `--debug` and `--trace` flags for `drone exec` so users can see Docker connection details, lifecycle events, and runtime errors when running pipelines locally."  
_(Primary repo: CLI)_

---

## 2. User-Facing Changes

> Derived from `spec.md` User Journeys and Functional Requirements.

- **Journeys covered in this repo**:
  - [UJ-01 – Debugging a standard pipeline run]
  - [UJ-02 – Deep tracing of runtime failures]
- **Flags / Interfaces**:
  - `drone exec --debug`
  - `drone exec --trace`
- **Behavior**:
  - [Bullet(s) describing behavior visible to users or calling systems]

If this repo is a dependency (e.g., engine, library), describe the user-facing effect **indirectly** (how this repo change supports the primary UX).

---

## 3. Internal Changes (This Repo)

> Derived from `plan.md` components and `tasks.md` completion.

### 3.1 Components / Areas Touched

- **Components** (from plan.md):
  - [CMP-xx – Name, short responsibility]
- **Key files / modules**:
  - `path/to/file1.go` – [short description]
  - `path/to/file2.go` – [short description]

### 3.2 Tasks Completed

> Summarize the completed tasks for this repo from `tasks.md`.

- [x] [Task ID / label] – [Short description]
- [x] [Task ID / label] – [Short description]
- [x] [Task ID / label] – [Short description]

If some tasks remain for the global feature but are **out of scope for this repo**, call that out explicitly.

---

## 4. Verification

> Derived from `verify.md` and `plan.md` Testing & Verification Strategy.

### 4.1 Golden Paths

- **GP-xx** – [Name] – ✅/❌  
  - How it was executed: [Command, script, or scenario]
  - Evidence: [Where to find logs/screenshots/test runs]

### 4.2 Failure Scenarios

- **EF-xx** – [Name] – ✅/❌  
  - How it was simulated: [Steps]
  - Observed behavior: [Summary]

### 4.3 Regression Check

- [ ] Existing behavior preserved for users with no flags or defaults.  
- [ ] No unexpected log noise or behavioral changes when feature is disabled.

---

## 5. Compatibility, Versioning & Migration

> Especially important when multiple repos are involved.

- **Version bump** (if applicable):
  - Before: [old-version]
  - After: [new-version]
- **Compatibility**:
  - [Statement about compatibility with existing callers / CLIs / services]
- **Breaking changes**:  
  - [Yes/No; if Yes, describe impact and migration steps]
- **Migration / Rollout Plan**:
  - [Steps for rolling out across repos/services]

---

## 6. Risks & Follow-Ups

> Derived from `plan.md` Risks/Trade-offs/TBDs and `verify.md` open issues.

- **Known Risks**:
  - [Risk-xx: description]
- **Open Issues / TODOs**:
  - [BUG or NEEDS CLARIFICATION entries that affect this repo]
- **Future Work**:
  - [Out-of-scope improvements tracked elsewhere]

---

## 7. Checklist

- [ ] `spec.md` for `[feature-id]` is up-to-date and aligned with this PR.
- [ ] `plan.md` for `[feature-id]` still accurately reflects the implementation.
- [ ] `tasks.md` shows all in-scope work for this repo as completed.
- [ ] `verify.md` has entries for all relevant Golden Paths and failure scenarios.
- [ ] Version bump (if any) is consistent with dependent/primary repos.
- [ ] This PR description can be understood without reading chat logs.
