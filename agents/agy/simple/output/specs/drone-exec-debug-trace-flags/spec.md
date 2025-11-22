# Spec: `drone exec` `--debug` and `--trace` flags

## 1. Context

### 1.1 Repositories

- **CLI**: `github.com/harness/drone-cli`
  - Provides the `drone` binary and the `drone exec` subcommand.
- **Docker runner engine**: `github.com/drone-runners/drone-runner-docker`
  - Implements the engine responsible for compiling and executing pipelines against the local Docker daemon.

### 1.2 Current behavior (high-level)

- `drone exec` behaves as a **local runner**:
  - Reads `.drone.yml` (or explicit pipeline file) in the current workspace.
  - Compiles the pipeline spec.
  - Executes it directly on the machine’s Docker daemon using the Docker Engine API.
- It uses the Docker client initialized from environment variables (e.g. `DOCKER_HOST`, `DOCKER_CERT_PATH`, etc.) via `client.NewClientWithOpts(client.FromEnv)`.
- The engine lifecycle is:
  1. **Connect**: create Docker client.
  2. **Setup**: create network and volumes.
  3. **Run**: for each step, pull image, create container, start, stream logs, wait.
  4. **Destroy**: stop and remove containers, remove volumes and network.

### 1.3 Problem

- When Docker-related issues occur (misconfigured `DOCKER_HOST`, TLS problems, missing daemon, GPU / runtime misconfiguration, etc.), the current `drone exec` UX provides limited, inconsistent or hard-to-correlate output.
- There is no **first-class, discoverable** way (`--debug`, `--trace`) to:
  - Turn on verbose logging for CLI and engine.
  - See key connection parameters and lifecycle events.
  - Inspect where exactly a run fails (connect, setup, container create, container start, logs, destroy).

### 1.4 Goal

Introduce **two explicit flags** for `drone exec`:

- `--debug`
- `--trace`

and wire them through the CLI, config, environment variables and engine so that they:

1. Are visible and discoverable in `drone exec --help`.
2. Activate **structured and actionable logs** across the CLI and Docker engine.
3. Make it straightforward to diagnose Docker connectivity/runtime issues locally and in CI (e.g. on builder VMs).


## 2. Objectives & Non-objectives

### 2.1 Objectives

1. **CLI UX**
   - Add `--debug` and `--trace` flags to `drone exec`.
   - Ensure they are documented in help text and any relevant README / usage docs.

2. **Configuration flow**
   - Extend the internal options/config structs for `drone exec` to carry `Debug bool` and `Trace bool`.
   - Map flags → config → environment → logging in a consistent, testable way.
   - Make sure these values can be consumed both by the CLI and the Docker engine.

3. **Logging behavior**
   - When `--debug` is enabled:
     - Log high-level lifecycle events and key parameters (Docker host, API version, network name, container IDs, step names, etc.).
   - When `--trace` is enabled:
     - Log **more granular events** and internal details (e.g. container create/start parameters, timing, network creation details, and any available error payloads from Docker).
   - If both are enabled, `--trace` implies `--debug` (or behaves as strictly more verbose than `--debug`).

4. **Engine integration**
   - Ensure `drone-runner-docker` receives `Debug` / `Trace` signals.
   - Add logging around:
     - Docker client creation (`NewEnv`).
     - `Setup` (network + volumes).
     - `Run` (per-step container create/start/log/exit).
     - `Destroy` (cleanup of containers, volumes, network).

5. **Testability**
   - Add unit tests around flag parsing and config wiring.
   - Provide manual test scenarios and commands for quickly verifying debug/trace behavior under different Docker conditions.

### 2.2 Non-objectives

- Do **not** implement a new logging backend or logging format (stick to existing logger abstractions used by drone-cli / drone-runner-docker).
- Do **not** change the core pipeline semantics or the public YAML schema.
- Do **not** introduce a new configuration file format for logging; flags + existing env vars are sufficient.
- Do **not** refactor the entire engine package; confine code changes to the minimum necessary for wiring and logging. (Refactors may be handled as a separate `/refactor` workflow if needed.)


## 3. Desired UX & CLI Contract

### 3.1 `drone exec --help`

`drone exec` help should clearly surface the new flags, for example:

```text
Usage: drone exec [options]

Options:
  --pipeline string   name of the pipeline to execute
  --env-file string   load environment variables from a file
  --trusted           run pipeline in trusted mode
  --debug             enable debug logging for CLI and Docker engine
  --trace             enable trace-level logging (more verbose than --debug)
  ... (other existing options)
```

### 3.2 Flag semantics

- `--debug`:
  - Enables **debug-level logs** in both the CLI and Docker engine.
  - Intended for regular troubleshooting by engineers.
  - Typical information:
    - Effective `DOCKER_HOST` and selected API version.
    - Engine driver (`bridge` or `nat`).
    - Created network ID / name.
    - Created container IDs per step.
    - Step lifecycle: `pull`, `create`, `start`, `logs`, `wait`, `exit status`.

- `--trace`:
  - Implies `--debug`.
  - Enables more **fine-grained logs**, suitable for deep debugging or upstream issue reports.
  - Typical information:
    - Timing information (duration of `pull`, `create`, `start`, `wait`).
    - Detailed Docker error payloads when operations fail.
    - Potentially additional internal state transitions (but still avoiding secrets).

### 3.3 Mapping to environment / config

- Internally, we define per-run options struct (name is illustrative):

```go
type ExecOptions struct {
    // existing fields
    // ...

    Debug bool
    Trace bool
}
```

- CLI flag resolution:
  - `--debug` sets `ExecOptions.Debug = true`.
  - `--trace` sets `ExecOptions.Trace = true` (and may implicitly set `Debug = true`).

- Environment mapping (high-level contract):
  - `ExecOptions.Debug` → `DRONE_DEBUG= true|false`
  - `ExecOptions.Trace` → `DRONE_TRACE= true|false`

- The engine layer and logger should read these values and adjust logging accordingly.


## 4. System Behavior with Debug/Trace Enabled

### 4.1 Docker client initialization

When creating a Docker client (in `drone-runner-docker/engine.NewEnv` or equivalent), the debug logs should include:

- `DOCKER_HOST` value (or default if unset).
- Whether TLS is enabled (e.g. CERT path detected, API scheme).
- Docker client API version (if available).

Examples (pseudologs):

```text
[DEBUG] docker client: host=unix:///var/run/docker.sock tls=false api_version=1.45
[DEBUG] docker client: host=tcp://docker.example:2376 tls=true api_version=1.41
```

On failure (e.g. cannot connect to daemon), trace-level logs should surface the underlying error details:

```text
[TRACE] docker client: dial error: dial unix /var/run/docker.sock: connect: no such file or directory
[DEBUG] failed to initialize docker client (see trace logs above)
```

### 4.2 Setup phase

During `Setup`, we expect logs such as:

- Network creation:

```text
[DEBUG] setup: creating network id=drone-<hash> driver=bridge
[TRACE] setup: network create request payload=...
[DEBUG] setup: network created id=<network-id>
```

- Volume creation (if applicable):

```text
[DEBUG] setup: creating volume name=drone-workspace-<hash>
[DEBUG] setup: volume created name=drone-workspace-<hash>
```

On errors, trace logs should capture Docker’s response:

```text
[TRACE] setup: network create error: {"message":"could not find driver"}
[DEBUG] setup: failed to create network: could not find driver
```

### 4.3 Run phase (per step)

For each step in the pipeline:

- Image handling:

```text
[DEBUG] step=build image=pytorch/pytorch:2.7.0-cuda12.8-cudnn9-devel action=pull
[TRACE] step=build image pull started
[TRACE] step=build image pull completed duration=12.34s
```

- Container creation and start:

```text
[DEBUG] step=build action=create container_id=<id>
[TRACE] step=build container config image=... env=[...] mounts=[...]
[DEBUG] step=build action=start container_id=<id>
```

- Logs and exit:

```text
[DEBUG] step=build action=logs streaming=true
[TRACE] step=build stdout="[preflight] GPU info via torch: ..."
[DEBUG] step=build action=wait status=0 duration=34.56s
```

On failures (e.g. cannot start container):

```text
[TRACE] step=build container start error: {"message":"failed to create shim task"}
[DEBUG] step=build failed: failed to start container (see trace logs for details)
```

### 4.4 Destroy phase

On teardown:

```text
[DEBUG] destroy: stopping containers count=3
[TRACE] destroy: container kill id=<id> signal=SIGKILL
[DEBUG] destroy: removing containers
[DEBUG] destroy: removing network id=<network-id>
```

On failures, trace logs include Docker payloads; debug logs summarize the failure.


## 5. Error Handling & User-Facing Messages

### 5.1 General principles

- **Do not leak secrets** in debug/trace logs.
- Prefer **actionable** messages over raw errors, e.g.:

```text
Error: failed to connect to Docker daemon.

Hints:
  - Check that Docker is running.
  - Check DOCKER_HOST and TLS settings.
  - Re-run with --debug or --trace for details.
```

- Use debug/trace logs for deeper context (host, API version, dial errors, etc.).

### 5.2 Typical scenarios

1. **Docker daemon not running**
   - User runs: `drone exec --debug`.
   - Expected behavior:
     - Debug output clearly shows the attempted `DOCKER_HOST`.
     - CLI exits non-zero with a clear message.
     - Optional hint: `docker ps` as a sanity check.

2. **Misconfigured DOCKER_HOST**
   - Wrong socket path or TCP host.
   - Expected behavior:
     - Trace logs show dial error (`connect: no such file or directory`, `connection refused`, etc.).
     - Debug logs summarize failure: "failed to connect to docker host ...".

3. **TLS misconfiguration**
   - Missing certs, wrong paths.
   - Expected behavior:
     - Trace logs show TLS handshake errors.
     - Debug logs mention TLS enabled + that handshake failed.

4. **GPU / runtime mismatches** (e.g. `nvidia-container-runtime` issues)
   - With `--debug`/`--trace`, step logs should show:
     - Container create/start requests and any returned error message.
     - Enough context for the user to realize it’s a runtime/driver issue (and not a YAML or CLI problem).


## 6. Edge Cases & Constraints

### 6.1 Flag precedence & combinations

- If both `--debug` and `--trace` are passed:
  - `Trace` supersedes `Debug` or is treated as strictly more verbose.
- If environment variables `DRONE_DEBUG` / `DRONE_TRACE` are already set:
  - CLI flags should have **higher priority** than environment defaults.
  - Behavior example:
    - `DRONE_DEBUG=false` in env, `drone exec --debug` → debug mode is **enabled**.
    - `DRONE_TRACE=false` in env, `drone exec --trace` → trace mode is **enabled**.

### 6.2 Backward compatibility

- Without passing the new flags and without env vars, behavior must remain compatible with current release (logging verbosity unchanged).
- Existing scripts/pipelines invoking `drone exec` without flags must continue to work.

### 6.3 CI usage

- The new flags must work equally well:
  - On local development machines.
  - On CI builder VMs (e.g., GCE DLVMs running `drone exec` for GPU builds).
- They must not introduce interactive prompts or any behavior requiring TTY.


## 7. Acceptance Criteria

1. **CLI UX**
   - `drone exec --help` shows `--debug` and `--trace` with clear descriptions.
   - Running `drone exec --debug` produces more verbose logs than the baseline.
   - Running `drone exec --trace` produces even more detailed logs.

2. **Configuration & environment**
   - There are unit tests confirming:
     - Flags are parsed and stored in the options struct.
     - Options are mapped to `DRONE_DEBUG` / `DRONE_TRACE` env vars or equivalent configuration consumed by the engine/logging.
     - CLI flags override env defaults.

3. **Engine logging**
   - With `--debug` enabled, logs include:
     - Effective `DOCKER_HOST`, TLS on/off, API version.
     - Network creation, container IDs per step, and high-level lifecycle messages.
   - With `--trace` enabled, logs additionally include:
     - Detailed error payloads from Docker on failure.
     - Timing information and low-level lifecycle details.

4. **Error paths**
   - In scenarios like stopped Docker daemon, wrong `DOCKER_HOST`, or TLS misconfig:
     - The CLI exits with a clear, actionable error message.
     - Debug/trace logs surface underlying errors and context.

5. **Manual validation checklist**

   At minimum, the following manual tests are documented and pass:

   - **Happy path:**
     - Docker daemon running.
     - Run `drone exec --debug --pipeline <simple-pipeline>`.
     - Observe debug logs for connection, network creation, step lifecycle.
   - **Trace path:**
     - Same pipeline with `--trace` and confirm more granular logs.
   - **Stopped daemon:**
     - Stop Docker.
     - Run `drone exec --debug` and verify error and logs.
   - **Misconfigured DOCKER_HOST:**
     - Set `DOCKER_HOST` to an invalid socket.
     - Run `drone exec --trace` and verify that dial errors are logged in detail.


## 8. Open Questions

1. Should we expose an explicit `--log-level` flag separate from `--debug` / `--trace`, or keep the simple boolean flags for now?
2. Should trace logs include Docker API request/response bodies by default, or only summaries to avoid overly noisy logs and possible leakage of sensitive configuration?
3. Are there existing logging conventions (e.g. log field names, prefixes) in `drone-cli` / `drone-runner-docker` that we must strictly follow for consistency?
4. Do we need a way to route debug/trace logs to a file (or JSON) in addition to stdout/stderr for easier ingestion into systems like Google Cloud Logging?

