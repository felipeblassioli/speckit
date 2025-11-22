# Drone Exec Command Specification

## Goal Description
The `drone exec` command executes a local build of a pipeline defined in a `.drone.yml` file. It allows developers to run and debug pipelines locally without pushing to a remote server.

## Visual Requirements
As a CLI tool, the "visual" requirements pertain to the terminal output.

- **Standard Output**:
  - By default, logs from steps are streamed to stdout.
  - If `--pretty` is enabled, the output should be formatted with colors and structure for better readability.
  - If `--debug` or `--trace` is enabled, additional log information (debug/trace level) should be printed.
- **Error Reporting**:
  - Errors during compilation, linting, or execution should be printed to stderr.
  - If the pipeline fails, the command should exit with a non-zero status code.
  - A JSON dump of the state may be printed on error if debugging is needed (implied by `dump(state)` in code).

## Functional Requirements

### Command Structure
`drone exec [path/to/.drone.yml]`

- **Arguments**:
  - `path/to/.drone.yml`: Optional. Path to the pipeline configuration file. Defaults to `.drone.yml` if not provided.

### Flags

#### Pipeline Selection & Filtering
- `--pipeline`: Name of the pipeline to execute (useful if multiple pipelines are defined in the file). Defaults to 'default' if not specified.
- `--include`: List of step names to include. Only these steps will be executed.
- `--exclude`: List of step names to exclude. These steps will be skipped.
- `--resume-at`: Name of the step to resume execution from. Steps prior to this will be skipped.

#### Execution Environment
- `--clone`: Boolean. Enable the clone step. Defaults to `false` (local build mounts current directory).
- `--trusted`: Boolean. Mark the build as trusted.
- `--timeout`: Duration. Build timeout. Defaults to 1 hour.
- `--volume`: List of build volumes in `host:container` format.
- `--network`: List of external networks to attach to.
- `--privileged`: List of privileged plugins. Defaults to standard plugins (docker, acr, ecr, gcr, heroku).
- `--registry`: Path to a registry file.

#### Secrets & Environment
- `--secret-file`: Path to a file containing secrets (key=value).
- `--env-file`: Path to a file containing environment variables (key=value).
- `--netrc-username`, `--netrc-password`, `--netrc-machine`: Netrc credentials.

#### Trigger Context (Metadata)
- `--branch`: Branch name. Sets `DRONE_BRANCH`, `DRONE_COMMIT_BRANCH`, `DRONE_TARGET_BRANCH`.
- `--event`: Build event name (push, pull_request, etc). Sets `DRONE_EVENT`.
- `--instance`: Instance hostname. Sets `DRONE_SYSTEM_HOST`, `DRONE_SYSTEM_HOSTNAME`.
- `--repo`: Git repository name (e.g., `octocat/hello-world`). Sets `DRONE_REPO`.
- `--deploy-to`: Deployment target. Sets `DRONE_DEPLOY_TO`.
- `--ref`: Git reference. Sets `DRONE_COMMIT_REF`.
- `--sha`: Git SHA. Sets `DRONE_COMMIT_SHA`.

#### Output & Debugging
- `--pretty`: Enable pretty printing.
- `--debug`: Enable debug logging (`DRONE_DEBUG`).
- `--trace`: Enable trace logging (`DRONE_TRACE`).

### Environment Variables
- The command inherits environment variables from the host.
- `DRONE_` prefixed variables from the host are passed to the pipeline environment.
- Variables defined in `--env-file` are loaded.

### Execution Logic
1.  **Load Configuration**: Read the `.drone.yml` file.
2.  **Environment Substitution**: Substitute variables in the config file using host environment and provided env file.
3.  **Parse & Lint**: Parse the manifest and lint the pipeline.
4.  **Compile**: Compile the pipeline into an intermediate representation (Spec).
    - If `--clone` is false (default), the current working directory is mounted.
    - Labels are added for step identification.
5.  **Filter Steps**: Apply `--include`, `--exclude`, and `--resume-at` filters.
6.  **Execute**: Run the pipeline steps using the Docker engine.
7.  **Signal Handling**: Listen for OS signals to gracefully terminate the build.

## Edge Cases
- **Missing Config File**: If the specified (or default) config file does not exist, return an error.
- **Invalid Config**: If the config file is invalid YAML or fails linting, return an error.
- **Pipeline Not Found**: If the specified `--pipeline` does not exist in the manifest, return an error.
- **Step Not Found**: If `--resume-at`, `--include`, or `--exclude` reference non-existent steps, the behavior depends on implementation (currently `resume-at` just iterates, `include`/`exclude` might just not match anything).
- **Timeout**: If the build exceeds the timeout, it is cancelled.
- **Signal Interruption**: If the user sends a signal (e.g., Ctrl+C), the build is cancelled and the process terminates.
- **Docker Connectivity**: Requires a running Docker daemon. Failure to connect will result in an error.
