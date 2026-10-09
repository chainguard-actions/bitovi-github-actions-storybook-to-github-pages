<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Build' step directly interpolates `${{ inputs.install_command }}` and `${{ inputs.build_command }}` inside a `run:` shell block. These are user-controlled inputs that are template-substituted by the Actions runner before the shell processes them, allowing an attacker who calls this composite action to inject arbitrary shell commands (e.g., supplying `; malicious-command` as the input value). The fix is to pass these values through environment variables and invoke them safely (e.g., via `env:` mapping and then executing `$INSTALL_COMMAND` with proper quoting), or restrict inputs to a known-safe allowlist.

Locations:

- `action.yaml:41`
- `action.yaml:42`

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

Fixed script injection in action.yaml's 'Build' step. Moved `${{ inputs.install_command }}` and `${{ inputs.build_command }}` out of the `run:` block and into the step's `env:` block as `INSTALL_COMMAND` and `BUILD_COMMAND`. Each command is then written to a temporary file via `printf '%s\n' "$VAR" > tmpfile` and executed with `bash -eo pipefail tmpfile` to preserve the errexit/pipefail semantics of the parent shell. Temp files are cleaned up after execution. This prevents attacker-controlled input values from being template-substituted directly into the shell script by the Actions runner.

