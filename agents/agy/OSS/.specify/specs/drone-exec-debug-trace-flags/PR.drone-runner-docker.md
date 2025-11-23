---
description: "Pull Request description for drone-runner-docker"
---
# [drone-exec-debug-trace-flags] – Implement structured debug/trace logging in Docker engine

**Feature ID**: drone-exec-debug-trace-flags  
**Repo**: drone-runner-docker  
**Spec**: .specify/specs/drone-exec-debug-trace-flags/spec.md  
**Plan**: .specify/specs/drone-exec-debug-trace-flags/plan.md  
**Tasks**: .specify/specs/drone-exec-debug-trace-flags/tasks.md  
**Verify**: .specify/specs/drone-exec-debug-trace-flags/verify.md  
**Related Repos**: drone-cli

---

## 1. Summary

- **Problem**: The Docker engine implementation lacked sufficient logging to diagnose connectivity and runtime issues, making it hard to troubleshoot failures in `drone exec`.
- **Goal**: Implement structured and actionable logging across the Docker engine lifecycle (connect, setup, run, destroy) to support the new `--debug` and `--trace` flags.
- **Scope (This Repo)**: This PR enhances the `engine` package to emit debug and trace logs, including host configuration, network/volume IDs, container config, and raw error payloads.

---

## 2. User-Facing Changes

- **Journeys covered in this repo**:
  - UJ-01 – Debugging a Standard Pipeline Run
  - UJ-02 – Deep Tracing of Runtime Failures
- **Behavior**:
  - **Debug Mode**: Logs connection details (Host, API version), network creation, and step lifecycle events.
  - **Trace Mode**: Logs detailed container configuration (Image, Env, Mounts), timing information for pulls/waits, and raw Docker error payloads.
  - **Safety**: Secrets are not leaked in logs.

---

## 3. Internal Changes (This Repo)

### 3.1 Components / Areas Touched

- **Components**:
  - CMP-02 [Docker Engine]: `drone-runner-docker/engine`
- **Key files / modules**:
  - `engine/engine.go`: Enhanced `NewEnv`, `Setup`, `Run`, and `Destroy` methods to support structured logging.

### 3.2 Tasks Completed

- [x] P1-T02 [P] [Engine] Enhance `NewEnv` in `drone-runner-docker/engine/engine.go` to log TLS status and detailed Host info.
- [x] P1-T03 [Engine] Audit `Setup` and `Destroy` debug logs to ensure Network/Volume IDs are consistently logged.
- [x] P2-T01 [Engine] Update `NewEnv` to log detailed dial/connection errors in Trace mode before returning.
- [x] P2-T02 [Engine] Update `create` (in `engine.go`) to log container configuration (Image, Env, Mounts) in Trace mode.
- [x] P2-T03 [Engine] Ensure `Run` logs timing information for Pull and Wait operations in Trace mode.
- [x] P2-T04 [Engine] Capture and log raw Docker error payloads in `Setup` and `Run` failure paths.
- [x] XCC-01 [Safety] Review all new log statements to ensure no secret values are logged.
- [x] XCC-02 [Consistency] Use consistent field names across all log entries.

---

## 4. Verification

### 4.1 Golden Paths

- **GP-01** – `drone exec --debug` – ✅
  - How it was executed: Ran `drone exec --debug` on a simple pipeline.
  - Evidence: Logs showed `docker client: host=...`, `setup: creating network`, `step=... action=create`.

- **GP-02** – `drone exec --trace` – ✅
  - How it was executed: Ran `drone exec --trace` on a simple pipeline.
  - Evidence: Logs showed image pull duration and container config details.

### 4.2 Failure Scenarios

- **EF-01** – Connection Failure – ✅
  - How it was simulated: Set `DOCKER_HOST=tcp://invalid:2376` and ran with `--trace`.
  - Observed behavior: Logs showed `dial tcp ...` error.

- **EF-02** – Runtime Failure – ✅
  - How it was simulated: Ran a pipeline with a command that fails immediately.
  - Observed behavior: Logs showed exit code and wait duration.

### 4.3 Regression Check

- [x] Existing behavior preserved for users with no flags or defaults.
- [x] No unexpected log noise or behavioral changes when feature is disabled.

---

## 5. Compatibility, Versioning & Migration

- **Compatibility**: Backward compatible.
- **Breaking changes**: No.

---

## 6. Risks & Follow-Ups

- **Known Risks**:
  - Log volume in trace mode might be high.
- **Open Issues / TODOs**:
  - None.

---

## 7. Checklist

- [x] `spec.md` for `drone-exec-debug-trace-flags` is up-to-date and aligned with this PR.
- [x] `plan.md` for `drone-exec-debug-trace-flags` still accurately reflects the implementation.
- [x] `tasks.md` shows all in-scope work for this repo as completed.
- [x] `verify.md` has entries for all relevant Golden Paths and failure scenarios.
- [x] This PR description can be understood without reading chat logs.
