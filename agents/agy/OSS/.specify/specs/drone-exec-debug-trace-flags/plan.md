# Plan: Drone Exec Debug & Trace Flags

**Feature ID**: drone-exec-debug-trace-flags
**Location**: .specify/specs/drone-exec-debug-trace-flags/plan.md
**Spec**: spec.md
**Status**: Draft

> ROLE OF THIS FILE
> - Translate spec **WHAT/WHY** into a concrete technical approach.
> - Provide enough structure for `/execute` and `/verify` to coordinate agents.
> - Stay high-level; push large code/config into separate artifacts.

---

## 1. Design Summary

- **Problem Restatement**: The current `drone exec` command lacks sufficient visibility into Docker engine interactions, making it difficult to diagnose connectivity (TLS, Host) and runtime (GPU, Shim) issues.
- **Overall Approach**: Enhance the existing `drone-cli` flags and `drone-runner-docker` engine to implement a structured logging strategy. We will leverage the existing `Debug` and `Trace` options but expand their coverage to include critical lifecycle events, configuration details, and raw error payloads that are currently missing.
- **Interface Shape**: CLI flags (`--debug`, `--trace`) mapped to internal `engine.Opts`.
- **Key Dependencies**: `drone-runner-docker` (local workspace) for the engine implementation.

---

## 2. Technical Shape

### 2.1 Components

- **CMP-01 [CLI Entrypoint]**: `drone-cli/drone/exec`
  - Responsibility: Parse `--debug` and `--trace` flags, configure global logger level, and initialize the engine with these options.
  - Status: Largely implemented, needs verification of flag precedence and env var overrides.

- **CMP-02 [Docker Engine]**: `drone-runner-docker/engine`
  - Responsibility: Execute the pipeline steps against the Docker daemon.
  - Change: Enhance `NewEnv`, `Setup`, `Run`, and `Destroy` methods to emit structured debug/trace logs as defined in the spec.

### 2.2 Data & Contracts

- **Domain Entities**:
  - `ExecOptions` (CLI side): Carries `Debug` and `Trace` bools.
  - `engine.Opts` (Engine side): Receives `Debug` and `Trace` bools.
- **Core Contracts**:
  - **CTR-01**: `engine.NewEnv(opts)` must log connection details (Host, API version) and failure reasons.
  - **CTR-02**: `engine.Run(...)` must log step lifecycle events (pull, create, start, wait) with timing and error payloads.

### 2.3 Execution Environment

- **Runtime**: CLI tool running on user's machine or CI builder.
- **Deployment Surface**: Binary distribution (`drone`).
- **Notable Constraints**: Must not leak secrets in logs.

---

## 3. Story-by-Story Implementation Strategy

### UJ-01 – Debugging a Standard Pipeline Run (Priority: P1)

- **Goal**: User runs `drone exec --debug` and sees high-level lifecycle events (Host, Network, Container IDs).
- **Dependencies**: CMP-01, CMP-02.

**Implementation Steps (High-Level Tasks)**

- [ ] P1-T01 [P] [CLI] Verify and fix flag precedence in `drone-cli/drone/exec/flags.go` (ensure CLI overrides Env).
- [ ] P1-T02 [P] [Engine] Enhance `NewEnv` in `drone-runner-docker/engine/engine.go` to log TLS status and detailed Host info.
- [ ] P1-T03 [Engine] Audit `Setup` and `Destroy` debug logs to ensure Network/Volume IDs are consistently logged.

**Verification Hooks**

- [ ] `/verify` can run `drone exec --debug` on a hello-world pipeline.
- [ ] Logs must contain: `docker client: host=...`, `setup: creating network`, `step=... action=create`.

---

### UJ-02 – Deep Tracing of Runtime Failures (Priority: P2)

- **Goal**: User runs `drone exec --trace` and sees low-level details (timing, config, raw errors) for deep debugging.
- **Dependencies**: P1-T02 (Engine logging foundation).

**Implementation Steps (High-Level Tasks)**

- [ ] P2-T01 [Engine] Update `NewEnv` to log detailed dial/connection errors in Trace mode before returning.
- [ ] P2-T02 [Engine] Update `create` (in `engine.go`) to log container configuration (Image, Env, Mounts) in Trace mode.
- [ ] P2-T03 [Engine] Ensure `Run` logs timing information for Pull and Wait operations in Trace mode.
- [ ] P2-T04 [Engine] Capture and log raw Docker error payloads in `Setup` and `Run` failure paths.

**Verification Hooks**

- [ ] `/verify` scenario: Run with invalid `DOCKER_HOST` and check for `dial error` in trace logs.
- [ ] `/verify` scenario: Run with non-existent image (force pull error) and check for detailed error payload.

---

## 4. Foundation & Cross-Cutting Work

### 4.1 Foundations (Blocking)

- None. Existing logging infrastructure (`github.com/drone/runner-go/logger`) is sufficient.

### 4.2 Cross-Cutting Concerns

- [ ] XCC-01 – **Secret Safety**: Review all new log statements to ensure no secret values (from `engine.Secret` or `step.Auth`) are logged.
- [ ] XCC-02 – **Consistency**: Use consistent field names (`step=`, `action=`, `duration=`) across all log entries.

---

## 5. Testing & Verification Strategy

### 5.1 Golden Paths

- **GP-01**: `drone exec --debug` (Success)
  - Run a simple pipeline.
  - Assert logs show: Host info, Network creation, Container Start/Wait, Network removal.
- **GP-02**: `drone exec --trace` (Success)
  - Run a simple pipeline.
  - Assert logs show: Image pull duration, Container config details.

### 5.2 Edge / Failure Scenarios

- **EF-01**: Connection Failure
  - Set `DOCKER_HOST=tcp://invalid:2376`.
  - Run `drone exec --trace`.
  - Assert logs show: `dial tcp ...` error.
- **EF-02**: Runtime Failure
  - Run a pipeline with a command that fails immediately.
  - Assert logs show: Exit code and wait duration.

### 5.3 Automation Surface

- [ ] Use `go test` for unit tests in `drone-cli` (flag parsing).
- [ ] Use `drone exec` (manual or scripted) for integration verification.

---

## 6. Risks, Trade-offs, and TBDs

- **Risk-01**: Log volume. Trace logs might be very verbose.
  - *Mitigation*: Ensure `Trace` is strictly opt-in and separate from `Debug`.
- **Risk-02**: Secret leakage in container config logging.
  - *Mitigation*: Explicitly sanitize or exclude `Env` vars that match secret patterns if logging full config.

**TBDs / Follow-Ups**

- [NEEDS CLARIFICATION: Should trace logs include Docker API request/response bodies by default?] -> *Assumption: No, only summaries and error payloads for now to avoid massive noise.*

---

## 7. Plan Checklist (Pre-/execute Gate)

### 7.1 Alignment & Simplicity

- [x] Every item in this plan traces back to `../spec.md` for `drone-exec-debug-trace-flags`.
- [x] No speculative "maybe later" features are planned here.
- [x] The number of components is minimal for the problem.
- [x] Any intentional complexity is listed in **Risks / Trade-offs**.

### 7.2 Clarity for Agents

- [x] Each high-level task clearly names its scope (files/modules/areas).
- [x] Parallelizable tasks are marked with `[P]`.
- [x] Foundations that must precede other work are under **4.1 Foundations**.
- [x] `/verify` hooks exist for each P1 journey.

### 7.3 Readiness

- [x] All critical `[NEEDS CLARIFICATION]` items that block implementation are resolved.
- [x] This plan can be executed by independent agents without hidden context.
- [x] This plan is small enough to be readable in one sitting.
