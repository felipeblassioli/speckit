# Antigravity Spec Kit - Simple Playbook

The workflows in `.agent/workflows` are minimal if compared with the usual Spec Kit.

See `outputs/specs` for some actual artifacts I tried on [drone-cli](https://github.com/harness/drone-cli) to debug an issue.
OBS: `outputs/specs/drone-exec-debug-trace-flags/spec.md` was written with Chat GPT not` `/define <feature-name>`

## 1. Mental model of your Antigravity Spec Kit

From your workflow files:

* `/define` → creates **`specs/[feature_name]/spec.md`** (single rigorous spec with visual + functional + edge cases). 
* `/architect` → creates **`specs/[feature_name]/plan.md`** with:

  * Section 1: **Architecture** (files, data models).
  * Section 2: **Step-by-step** checklists (`- [ ]`) tagged `[Frontend]` / `[Backend]`. 
* `/execute` → reads `plan.md`, executes **unchecked** steps, and marks them `- [x]` as it goes. 
* `/refactor` → takes a directory, compares against `memory/constitution.md`, refactors to modern patterns **without changing business logic**, then runs tests. 
* `/verify` → spins the app, does visual + “happy path” checks, and produces a **Release Readiness Artifact**. 
* The README already states the canonical cycle: `/define → /architect → /execute → /verify`. 

So: you’ve implemented core Spec Kit semantics in Antigravity already. The only missing piece is “how do I *drive* this for different types of work?”.

---

## 2. Standard workflows by *type* of work

### 2.1. Add new features

**Goal:** Introduce new externally-visible behavior.

**Canonical flow**

1. **/define – Spec the new behavior**

   * Command (Manager View):

     ```text
     /define [feature-name]
     ```

   * In the message body, tell the agent:

     * Where the feature lives (paths).
     * Visual expectations (screenshots / terminal examples).
     * Functional behavior + edge cases.

   * Your `/define` workflow will:

     * Read `memory/constitution.md`.
     * Create/update `specs/[feature-name]/spec.md` with:

       * Visual Requirements
       * Functional Requirements
       * Edge Cases (in the same file, no extra checklist file). 

2. **/architect – Turn spec into a build plan**

   ```text
   /architect [feature-name]
   ```

   * It reads `specs/[feature-name]/spec.md` and writes `specs/[feature-name]/plan.md` with: 

     * **Section 1: Architecture**

       * Files to change / create.
       * Data models / structs / DTOs.
       * Integration points (APIs, env vars, flags, etc.).
     * **Section 2: Step-by-Step**

       * `- [ ] [Backend] ...`
       * `- [ ] [Frontend] ...`
   * This becomes the single source of truth for implementation.

3. **/execute – Actually build it**

   ```text
   /execute [feature-name]
   ```

   * The workflow:

     * Reads `specs/[feature-name]/plan.md`. 
     * Picks **unchecked** `- [ ]` steps.
     * Implements them in code.
     * Marks each finished step as `- [x]`.
   * You (human) can interleave:

     * Running `go test`, `npm test`, etc.
     * Adjusting plan.md if something ends up different.

4. **/verify – QA & release readiness**

   ```text
   /verify [feature-name]
   ```

   * For web/app features: spin app, screenshot, happy-path. 
   * For CLI features: you’ll adapt:

     * Run CLI commands.
     * Capture terminal output instead of screenshots.
   * It outputs a “Release Readiness Artifact” (summary of checks, matches/discrepancies).

5. **Optional /refactor (before or after)**

   * For big/brittle areas, run:

     ```text
     /refactor path/to/legacy/module
     ```

   * That performs style / structural improvements, respecting constitution & tests. 

---

### 2.2. Fix bugs

Bugs are “mini-features” whose spec is: **“how it should behave” vs “how it behaves today”**.

**Recommended flow**

1. **Reproduce + observe manually** (you, not the agent):

   * Capture commands, inputs, outputs, logs.

2. **/define – Bug spec**

   ```text
   /define [bug-id]-[short-name]
   ```

   In the body, provide:

   * **Context**:

     * Code paths: `drone/exec/exec.go`, `engine/engine.go`, etc.
     * Env: `DRONE_SERVER`, `DRONE_TOKEN`, `DOCKER_HOST`, host OS.
   * **Current behavior** (broken).
   * **Expected behavior**.
   * **Reproduction steps**.
   * Any logs / Docker errors.

   The result is a `specs/[bug-id-short-name]/spec.md` that formally describes the bug and the correct behavior.

3. **/architect – Patch plan**

   ```text
   /architect [bug-id]-[short-name]
   ```

   * Architecture section will enumerate:

     * Where to add logging / guards.
     * Which functions must change.
     * Which tests to add/extend.
   * Step-by-step becomes: reproduce, add test, patch, verify, etc.

4. **/execute – Implement the fix**

   ```text
   /execute [bug-id]-[short-name]
   ```

   * The agent executes unchecked items:

     * Add regression test.
     * Adjust logic.
     * Wire env variables, etc.

5. **/verify – Regression check**

   ```text
   /verify [bug-id]-[short-name]
   ```

   * Runs the “happy path” plus bug-specific repro flow and ensures it now behaves as per spec.

6. **/refactor** only after tests are green, if appropriate.

---

### 2.3. Refactor

Refactor = **change structure, not behavior**.

**Two patterns:**

1. **Pure refactor on a module**

   * Optionally `/define` a “behavior spec” if the module has no tests.

   * Run:

     ```text
     /refactor path/to/module
     ```

   * The workflow:

     * Audits code vs `constitution.md` (async/await, logging patterns, error handling, etc.).
     * Applies refactors without changing business logic.
     * Runs tests. 

2. **Refactor as part of feature/bug**

   * Run `/refactor` on the area before or after `/execute`, but anchored by a spec:

     * Before: clean up the surface area to make implementation less painful.
     * After: once behavior is locked and tests exist, clean up internals safely.

---

## 3. Applying this to your **drone-cli debug build**

You already have a spec-like file for `drone exec` including `--debug` / `--trace` flags. 
Let’s turn that into a first-class Spec Kit feature and run it through your workflows.

### 3.1. Choose a feature name

Something like:

* `drone-exec-debug-trace-flags`

This will map to:

* `specs/drone-exec-debug-trace-flags/spec.md`
* `specs/drone-exec-debug-trace-flags/plan.md`

### 3.2. /define – Spec the debug/trace behavior properly

Command in Antigravity:

```text
/define drone-exec-debug-trace-flags
```

Prompt body (rough sketch):

> Create or update `specs/drone-exec-debug-trace-flags/spec.md` for the `drone exec` CLI command in the `drone-cli` repository.
> Context:
>
> * Repos: `github.com/harness/drone-cli` (CLI) and `github.com/drone-runners/drone-runner-docker` (engine).
> * I want to add explicit `--debug` and `--trace` flags that:
>
>   * Map to `DRONE_DEBUG` and `DRONE_TRACE` env vars used by the runner and logger stack.
>   * Enable detailed logs of Docker client initialization, network creation, and container lifecycle when running `drone exec`.
> * Current high-level behavior and requirements are described in this spec (paste content of your existing spec.md here).
>
> Please:
>
> * Incorporate the existing spec content into the new spec file.
> * Make “Output & Debugging” and “Docker Connectivity” behavior explicit, including:
>
>   * What additional information must be logged when `--debug` is on (e.g., DOCKER_HOST, API version, network name, container IDs).
>   * What additional traces must be logged when `--trace` is on (e.g., HTTP calls to Docker daemon if feasible, or at least step boundaries).
> * Include clear **Edge Cases**:
>
>   * Docker daemon unreachable / wrong DOCKER_HOST.
>   * Missing or invalid TLS configuration.
>   * Pipeline parse error vs runtime Docker error.

The `/define` workflow will consolidate that into a **single** `spec.md` under your `specs/` tree. 

### 3.3. /architect – Plan the change in terms of files & steps

Then:

```text
/architect drone-exec-debug-trace-flags
```

Given your `architect.md` workflow, you want Antigravity to produce a **rigorous `plan.md`**: 

You can steer it with something like:

> Read `specs/drone-exec-debug-trace-flags/spec.md` and create `specs/drone-exec-debug-trace-flags/plan.md`.
>
> * Section 1 (**Architecture**) must list:
>
>   * Changes to `drone/exec/exec.go` (or equivalent command entrypoint).
>   * Changes to configuration struct(s) that hold `Debug` / `Trace` flags.
>   * How we propagate those flags to:
>
>     * The logger setup (so `DRONE_DEBUG` / `DRONE_TRACE` are honored).
>     * The `engine.NewEnv` / Docker client creation in `drone-runner-docker/engine/engine.go`.
>   * New or updated tests in both repos.
> * Section 2 (**Step-by-Step**) should use checkboxes `- [ ]` and tag everything `[Backend]`.
>
>   * Include steps for:
>
>     * Inspecting existing debug / trace flags.
>     * Wiring CLI flags → config → env vars → logger.
>     * Adding targeted logging around `client.NewClientWithOpts(client.FromEnv)` and `Setup/Run/Destroy` flow.
>     * Building and testing the new binary.
>     * Manual testing commands to exercise Docker connectivity errors.

Conceptually, you want `plan.md` to end up with something like (example structure, not literal):

```md
## 1. Architecture

- Update CLI flags in `cmd/drone/exec.go` to add `--debug` and `--trace`.
- Extend config struct `ExecOptions` with `Debug bool` and `Trace bool`.
- Map options to:
  - Env vars `DRONE_DEBUG`, `DRONE_TRACE`.
  - Logger setup used by `drone-runner-docker` (if shared).
- Add detailed logging in:
  - `engine.NewEnv` (Docker client connection and host info).
  - `Docker.Setup` (network create).
  - `Docker.Run` (container create/start/logs).
  - `Docker.Destroy` (cleanup).
- Add Go tests for:
  - CLI → options mapping.
  - Options → env vars / logger config mapping.

## 2. Step-by-Step

- [ ] [Backend] Audit current handling of `DRONE_DEBUG` / `DRONE_TRACE` in both repos.
- [ ] [Backend] Add `--debug` / `--trace` flags to `drone exec` CLI and map them to options.
- [ ] [Backend] Wire options to env vars and logger configuration.
- [ ] [Backend] Add structured logging around Docker client and engine lifecycle (respecting debug/trace).
- [ ] [Backend] Add unit tests for flag parsing and config propagation.
- [ ] [Backend] Build a local `drone` binary and smoke-test basic pipeline execution.
- [ ] [Backend] Run manual tests to simulate Docker connectivity issues (wrong DOCKER_HOST, stopped daemon) and observe debug/trace logs.
- [ ] [Backend] Update `README` / help text to document `--debug` and `--trace`.
```

Antigravity should generate something along those lines automatically once you’ve seeded the spec.

### 3.4. /execute – Do the actual Go changes

Now:

```text
/execute drone-exec-debug-trace-flags
```

Your `/execute` workflow will: 

* Load `specs/drone-exec-debug-trace-flags/plan.md`.
* Find unchecked `- [ ]` items.
* Implement them, marking as `- [x]` as it goes.

How to steer it in the prompt:

> Read `specs/drone-exec-debug-trace-flags/plan.md`.
> Execute the unchecked steps in order, updating the code in:
>
> * `drone-cli` repo (CLI flags, config, help text, tests).
> * `drone-runner-docker` repo if needed (logging around Docker client / engine).
>   After every 2–3 steps, run:
> * `go test ./...` in the relevant repo.
> * `go build ./cmd/drone` (or equivalent).
>   Update the checkboxes from `- [ ]` to `- [x]` once a step is complete and verified.

You, as the human, can:

* Pull the branch.
* Run:

  * `go test ./...`
  * `DRONE_DEBUG=true DRONE_TRACE=true drone exec --pipeline build-mm-wheels --env-file .env.wheels --trusted --pretty`
    to confirm that you see more detailed logs around Docker connectivity.

### 3.5. /verify – Make sure it actually helps you debug Docker

Then:

```text
/verify drone-exec-debug-trace-flags
```

Your `/verify` workflow is written for “visual + logic” QA, but it adapts fine to a CLI: 

You can instruct it:

> For `drone exec` in the `drone-cli` repo:
>
> * **Visual Audit**:
>
>   * Run `drone exec --help` and capture the section showing `--debug` and `--trace`.
>   * Run a sample `drone exec` pipeline with and without `--debug` / `--trace` and capture representative output blocks.
>   * Compare the output to the “Visual Requirements” and “Output & Debugging” sections of the spec.
> * **Logic Audit**:
>
>   * Happy path: run `drone exec` against a known-good Docker setup and ensure behavior matches the spec.
>   * Failure path: simulate a broken Docker daemon (`DOCKER_HOST` pointing to a dead socket, or Docker stopped) and ensure:
>
>     * The process fails with a clear message.
>     * `--debug` / `--trace` add the expected extra context.
> * Produce a Release Readiness Artifact summarizing:
>
>   * Cases tested (happy path + failure scenarios).
>   * Whether observed output matches spec.
>   * Any discrepancies.

That artifact gives you something concrete to attach to a PR (or keep locally).

### 3.6. /refactor – Clean up the engine/logging if needed

If, while doing this, you realize `engine/engine.go` or the logging stack is gnarly, you can run:

```text
/refactor engine
```

With instructions like:

> Refactor the `engine` package in `drone-runner-docker` for readability and consistency with `memory/constitution.md`:
>
> * Prefer structured logging via the shared logger abstraction.
> * Avoid duplicated error wrapping for Docker client failures.
> * Do NOT change business logic or behavior.
> * Run `go test ./...` in the repo to validate no regressions.

That keeps the debugging surface clean without changing semantics. 

---
