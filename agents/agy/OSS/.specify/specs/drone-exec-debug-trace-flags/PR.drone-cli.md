---
description: "Pull Request description for drone-cli"
---
# [drone-exec-debug-trace-flags] – Add debug and trace flags to drone exec

**Feature ID**: drone-exec-debug-trace-flags  
**Repo**: drone-cli  
**Spec**: .specify/specs/drone-exec-debug-trace-flags/spec.md  
**Plan**: .specify/specs/drone-exec-debug-trace-flags/plan.md  
**Tasks**: .specify/specs/drone-exec-debug-trace-flags/tasks.md  
**Verify**: .specify/specs/drone-exec-debug-trace-flags/verify.md  
**Related Repos**: drone-runner-docker

---

## 1. Summary

- **Problem**: When Docker-related issues occur (misconfigured `DOCKER_HOST`, TLS problems, missing daemon), the current `drone exec` UX provides limited, inconsistent, or hard-to-correlate output. There is no first-class way to turn on verbose logging.
- **Goal**: Expose `--debug` and `--trace` flags for `drone exec` so users can see Docker connection details, lifecycle events, and runtime errors when running pipelines locally.
- **Scope (This Repo)**: This PR implements the CLI entrypoint changes, including parsing the new flags and wiring them to the engine configuration.

---

## 2. User-Facing Changes

- **Journeys covered in this repo**:
  - UJ-01 – Debugging a Standard Pipeline Run
  - UJ-02 – Deep Tracing of Runtime Failures
- **Flags / Interfaces**:
  - `drone exec --debug`
  - `drone exec --trace`
- **Behavior**:
  - Passing `--debug` enables debug-level logging (showing host info, network creation).
  - Passing `--trace` enables trace-level logging (showing raw errors, timing info).
  - CLI flags override `DRONE_DEBUG` and `DRONE_TRACE` environment variables.

---

## 3. Internal Changes (This Repo)

### 3.1 Components / Areas Touched

- **Components**:
  - CMP-01 [CLI Entrypoint]: `drone-cli/drone/exec`
- **Key files / modules**:
  - `drone/exec/flags.go`: Added `debug` and `trace` flags and logic to configure the global logger.

### 3.2 Tasks Completed

- [x] P1-T01 [P] [CLI] Verify and fix flag precedence in `drone-cli/drone/exec/flags.go` (ensure CLI overrides Env).

---

## 4. Verification

### 4.1 Golden Paths

- **GP-01** – `drone exec --debug` – ✅
  - How it was executed: Ran `drone exec --debug` on a simple pipeline.
  - Evidence: Logs showed `docker client: host=...` and `setup: creating network`.

- **GP-02** – `drone exec --trace` – ✅
  - How it was executed: Ran `drone exec --trace` on a simple pipeline.
  - Evidence: Logs showed image pull duration and container config details.

### 4.2 Failure Scenarios

- **EF-01** – Connection Failure – ✅
  - How it was simulated: Set `DOCKER_HOST=tcp://invalid:2376` and ran with `--trace`.
  - Observed behavior: Logs showed `dial tcp ...` error.

### 4.3 Regression Check

- [x] Existing behavior preserved for users with no flags or defaults.
- [x] No unexpected log noise or behavioral changes when feature is disabled.

---

## 5. Compatibility, Versioning & Migration

- **Compatibility**: Backward compatible. Running without flags behaves exactly as before.
- **Breaking changes**: No.

---

## 6. Risks & Follow-Ups

- **Known Risks**:
  - None specific to CLI.
- **Open Issues / TODOs**:
  - None.

---

## 7. Checklist

- [x] `spec.md` for `drone-exec-debug-trace-flags` is up-to-date and aligned with this PR.
- [x] `plan.md` for `drone-exec-debug-trace-flags` still accurately reflects the implementation.
- [x] `tasks.md` shows all in-scope work for this repo as completed.
- [x] `verify.md` has entries for all relevant Golden Paths and failure scenarios.
- [x] This PR description can be understood without reading chat logs.
