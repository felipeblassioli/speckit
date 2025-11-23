# Spec: Drone Exec Debug & Trace Flags

**Feature ID**: drone-exec-debug-trace-flags
**Location**: .specify/specs/drone-exec-debug-trace-flags/spec.md
**Status**: Draft
**Input**: User description: "based on @[specs/drone-exec-debug-trace-flags/spec.md] to match the new workflow and template @[.specify/templates/spec.md]"

> ROLE OF THIS FILE
> - Capture **WHAT** and **WHY** for exactly one bounded change (feature, bugfix, or refactor).
> - Stay implementation-agnostic: no stacks, frameworks, file paths, or algorithms.
> - Surface ambiguity explicitly with `[NEEDS CLARIFICATION: …]` instead of guessing.

---

## 1. Context & Intent

- **Problem / Motivation**:
  - When Docker-related issues occur (misconfigured `DOCKER_HOST`, TLS problems, missing daemon, GPU/runtime misconfiguration), the current `drone exec` UX provides limited, inconsistent, or hard-to-correlate output.
  - There is no first-class, discoverable way to turn on verbose logging for the CLI and engine to inspect where exactly a run fails.
- **Target Users / Systems**:
  - Developers troubleshooting local pipeline runs.
  - CI systems (e.g., builder VMs) where diagnosing connectivity or runtime issues is critical.
- **Interface Mode**: CLI Command
- **Scope (In)**:
  - Add `--debug` and `--trace` flags to `drone exec`.
  - Wire flags to internal config and environment variables (`DRONE_DEBUG`, `DRONE_TRACE`).
  - Implement structured and actionable logging across the CLI and Docker engine for lifecycle events (connect, setup, run, destroy).
- **Scope (Out)**:
  - New logging backend or format (must use existing abstractions).
  - Changes to core pipeline semantics or public YAML schema.
  - New configuration file formats.
  - Major refactoring of the engine package beyond wiring logging.

> If UI exists, reference design assets (screenshots, mocks).
> If backend-only, express intent via contracts and observable outcomes.

---

## 2. User Journeys (Slice-by-Slice)

> Each journey should be an independently testable slice.

### UJ-01 – Debugging a Standard Pipeline Run (Priority: P1)

- **Narrative**: A developer runs a pipeline locally but suspects an issue with the Docker environment. They run `drone exec --debug` to see high-level lifecycle events.
- **Value**: Provides immediate visibility into connection parameters, network creation, and step execution status without overwhelming detail.
- **Independent Test**: Run a simple pipeline with `--debug` and verify logs contain host info, network ID, and step status.

**Acceptance Scenarios**

1. **Given** a running Docker daemon and a valid `.drone.yml`, **When** running `drone exec --debug`, **Then** logs MUST show `DOCKER_HOST`, API version, network creation ID, and per-step container IDs.
2. **Given** a stopped Docker daemon, **When** running `drone exec --debug`, **Then** the CLI exits with a clear error and debug logs show the connection failure reason.

---

### UJ-02 – Deep Tracing of Runtime Failures (Priority: P2)

- **Narrative**: A developer or CI system encounters an obscure failure (e.g., "failed to create shim task"). They run `drone exec --trace` to see raw Docker error payloads and timing details.
- **Value**: Enables diagnosis of complex runtime or driver issues (e.g., GPU mismatches) by surfacing low-level details.
- **Independent Test**: Run a pipeline with `--trace` (potentially with a forced failure) and verify logs contain detailed error payloads and timing info.

**Acceptance Scenarios**

1. **Given** a misconfigured `DOCKER_HOST` (e.g., invalid socket), **When** running `drone exec --trace`, **Then** logs MUST show the specific dial error (e.g., `connect: no such file or directory`).
2. **Given** a pipeline step that fails to start, **When** running `drone exec --trace`, **Then** logs MUST include the raw Docker error message/payload.

---

## 3. Functional Requirements

> Describe observable behavior only. Do **not** pick technologies.

- **FR-001**: System MUST accept `--debug` and `--trace` flags in the `drone exec` command.
- **FR-002**: `drone exec --help` MUST document these flags.
- **FR-003**: `--debug` MUST enable debug-level logging in both CLI and Docker engine, showing configuration (Host, TLS, API ver) and lifecycle events (Network/Volume creation, Container ID, Step status).
- **FR-004**: `--trace` MUST imply `--debug` and additionally enable trace-level logging, showing detailed error payloads, timing information, and low-level lifecycle details.
- **FR-005**: CLI flags MUST override environment variables (`DRONE_DEBUG`, `DRONE_TRACE`) if both are present.
- **FR-006**: System MUST NOT leak secrets in debug or trace logs.
- **FR-007**: On connection or runtime failure, the CLI MUST exit with a clear, actionable error message (e.g., hints to check Docker daemon).

### Key Entities (If data is involved)

- **ExecOptions**: Internal configuration struct carrying `Debug` and `Trace` booleans.
- **Docker Client**: The entity initialized with the debug/trace context.

---

## 4. Non-Functional & Constraints

> Tech-agnostic but measurable targets.

- **NF-001 Performance**: Enabling `--debug` SHOULD NOT introduce significant latency (< 100ms overhead) to the pipeline execution.
- **NF-002 Reliability**: Logging MUST NOT cause the pipeline to crash or alter its execution logic (side-effect free).
- **NF-003 Security / Privacy**: Logs MUST be sanitized of secrets (e.g., registry credentials, secret env vars).
- **NF-004 Observability**: Logs MUST be structured enough to be human-readable in stdout/stderr.
- **NF-005 Backward Compatibility**: Running without flags MUST behave exactly as the current release (no extra logs).

---

## 5. Edge Cases & Failure Modes

- **Flag Combinations**: If both `--debug` and `--trace` are provided, `--trace` takes precedence (includes all debug info + trace info).
- **Environment Variables**: `DRONE_DEBUG=false` in env but `--debug` flag passed -> Debug enabled.
- **Missing Docker Daemon**: Should fail gracefully with a clear message, not a panic or raw stack trace (unless in trace mode where stack trace might be acceptable/helpful, but user-facing error should be clean).
- **TLS Errors**: specific handshake errors should be visible in trace logs.

---

## 6. Open Questions & Assumptions

> Use `[NEEDS CLARIFICATION: …]` for questions. Use assumptions sparingly.

- [NEEDS CLARIFICATION: Should trace logs include Docker API request/response bodies by default, or only summaries?]
- [NEEDS CLARIFICATION: Do we need a way to route debug/trace logs to a file in addition to stdout?]
- [NEEDS CLARIFICATION: Are there specific existing logging conventions (field names) we must strictly follow?]

**Assumptions**

- Existing logging abstractions in `drone-cli` and `drone-runner-docker` support level-based logging or can be easily adapted.
- Users primarily consume these logs via stdout/stderr in their terminal or CI console.

---

## 7. Readiness Checklist (Spec)

> Self-test before `/architect` runs.

- [x] This spec describes exactly **one** bounded change for `drone-exec-debug-trace-flags`.
- [x] All critical journeys (P1, P2) are listed with independent tests.
- [x] Each FR/NF requirement is observable and testable.
- [x] Success for this feature is expressible via a small set of metrics (logs visible, flags working).
- [x] All known ambiguities are listed in **Open Questions**.
- [x] `[NEEDS CLARIFICATION]` markers exist only where information truly does not exist.
- [x] No implementation details (frameworks, DBs, file layouts) appear above.
- [x] This spec is stable enough to generate `.specify/specs/drone-exec-debug-trace-flags/plan.md`.
