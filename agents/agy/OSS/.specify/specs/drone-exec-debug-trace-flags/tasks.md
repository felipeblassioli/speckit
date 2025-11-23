# Tasks: Drone Exec Debug & Trace Flags

**Feature ID**: drone-exec-debug-trace-flags
**Spec**: spec.md
**Plan**: plan.md
**Status**: Completed

> ROLE OF THIS FILE
> - The single source of truth for "what to do next".
> - Agents update this file as they complete work.

---

## Phase 1: Debugging a Standard Pipeline Run (P1)

- [x] P1-T01 [P] [CLI] Verify and fix flag precedence in `drone-cli/drone/exec/flags.go` (ensure CLI overrides Env).
- [x] P1-T02 [P] [Engine] Enhance `NewEnv` in `drone-runner-docker/engine/engine.go` to log TLS status and detailed Host info.
- [x] P1-T03 [Engine] Audit `Setup` and `Destroy` debug logs to ensure Network/Volume IDs are consistently logged.

## Phase 2: Deep Tracing of Runtime Failures (P2)

- [x] P2-T01 [Engine] Update `NewEnv` to log detailed dial/connection errors in Trace mode before returning.
- [x] P2-T02 [Engine] Update `create` (in `engine.go`) to log container configuration (Image, Env, Mounts) in Trace mode.
- [x] P2-T03 [Engine] Ensure `Run` logs timing information for Pull and Wait operations in Trace mode.
- [x] P2-T04 [Engine] Capture and log raw Docker error payloads in `Setup` and `Run` failure paths.

## Phase 3: Cross-Cutting Concerns

- [x] XCC-01 [Safety] Review all new log statements to ensure no secret values (from `engine.Secret` or `step.Auth`) are logged.
- [x] XCC-02 [Consistency] Use consistent field names (`step=`, `action=`, `duration=`) across all log entries.
