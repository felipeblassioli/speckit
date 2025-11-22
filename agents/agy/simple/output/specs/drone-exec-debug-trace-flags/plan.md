# Plan: `drone exec` `--debug` and `--trace` flags

## 1. Architecture

### Changed Files

#### `drone-cli`
- **`go.mod`**: Add `replace` directive to use local `drone-runner-docker`.
- **`drone/exec/exec.go`**: Update `engine.NewEnv` call to pass `Debug` and `Trace` flags.
- **`drone/exec/flags_test.go`** (NEW): Add unit tests for flag parsing.

#### `drone-runner-docker`
- **`engine/engine.go`**:
    - Update `Opts` struct to include `Debug` and `Trace`.
    - Update `Docker` struct to store `Debug` and `Trace` state.
    - Update `NewEnv` to accept and use these options.
    - Add structured logging (Debug/Trace) in `NewEnv`, `Setup`, `Run`, and `Destroy` methods.

### Data Models
- **`engine.Opts`**: Add `Debug bool` and `Trace bool`.
- **`engine.Docker`**: Add `debug bool` and `trace bool` fields.

## 2. Step-by-Step

- [x] **[Backend]** Modify `drone-cli/go.mod` to add `replace github.com/drone-runners/drone-runner-docker => ../drone-runner-docker`.
- [ ] **[Backend]** Modify `drone-runner-docker/engine/engine.go`:
    - [x] Update `Opts` struct with `Debug` and `Trace`.
    - [x] Update `Docker` struct.
    - [x] Update `NewEnv` to populate `Docker` struct from `Opts`.
    - [x] Add logging to `NewEnv` (Client init details).
    - [x] Add logging to `Setup` (Network/Volume creation).
    - [x] Add logging to `Run` (Container create/start/logs/wait).
    - [x] Add logging to `Destroy` (Cleanup details).
- [x] **[Backend]** Modify `drone-cli/drone/exec/exec.go`:
    - [x] Pass `commy.Debug` and `commy.Trace` to `engine.NewEnv`.
- [x] **[Backend]** Create `drone-cli/drone/exec/flags_test.go`:
    - [x] Test `mapOldToExecCommand` to verify `--debug` and `--trace` flags are correctly mapped to `execCommand`.

## 3. Verification Plan

### Automated Tests
- **Unit Test**: Run `go test ./drone/exec/...` in `drone-cli` to verify flag parsing.
    ```bash
    cd drone-cli
    go test -v ./drone/exec/...
    ```

### Manual Verification
- **Setup**: Create a simple `.drone.yml` in a temp directory.
    ```yaml
    kind: pipeline
    type: docker
    name: default
    steps:
    - name: hello
      image: alpine
      commands:
      - echo hello world
    ```
- **Happy Path (Debug)**:
    - Run: `go run ./cmd/drone/main.go exec --debug .drone.yml` (assuming running from `drone-cli` root).
    - Verify: Output contains `[debug]` logs showing Docker client init, network creation, etc.
- **Happy Path (Trace)**:
    - Run: `go run ./cmd/drone/main.go exec --trace .drone.yml`
    - Verify: Output contains `[trace]` logs with more granular details.
- **Error Case (Stopped Docker)**:
    - Stop Docker Desktop/Daemon.
    - Run: `go run ./cmd/drone/main.go exec --debug .drone.yml`
    - Verify: Clear error message and debug logs showing connection failure.
