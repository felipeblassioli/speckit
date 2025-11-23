# Verification: Drone Exec Debug & Trace Flags

**Feature ID**: drone-exec-debug-trace-flags
**Spec**: spec.md
**Plan**: plan.md
**Status**: Verified

## Golden Paths

### GP-01: `drone exec --debug`
- [x] Run `drone exec --debug` on a simple pipeline.
- [x] Assert logs show: Host info, Network creation, Container Start/Wait.
  - Evidence:
    ```text
    DEBU[0000] docker client: host=default tls=false api_version=1.40
    DEBU[0000] setup: creating network id=drone-Xt1OYeCbBWMIAk8Ll1uC driver=bridge
    DEBU[0000] container_id=drone-7xLX1CQG3gsW4XnKZE8D       action=create step=hello step.name=hello
    ```

### GP-02: `drone exec --trace`
- [x] Run `drone exec --trace` on a simple pipeline.
- [x] Assert logs show: Image pull duration, Container config details.
  - Evidence:
    ```text
    TRAC[0001] image pull completed duration=1.801296417s    step=hello step.name=hello
    TRAC[0001] container config image=docker.io/library/alpine:latest mounts=[/Users/felipe.blassioli/personal/OSS/drone/drone-cli:/drone/src]  step=hello step.name=hello
    ```
