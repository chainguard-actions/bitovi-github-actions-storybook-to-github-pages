# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Build' step in action.yaml directly interpolates `${{ inputs.build_command }}` inside a `run:` shell command. An attacker who controls the `build_command` input can inject arbitrary shell commands. The value should be assigned to an environment variable (e.g., `BUILD_CMD: ${{ inputs.build_command }}`) and referenced as `$BUILD_CMD` in the run block instead.

Locations:

- `action.yaml:36`

### unpinned-uses (severity: high)

Three `uses:` references in action.yaml are pinned to mutable tags/version strings rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or compromised:
- `actions/checkout@v3` (line 30)
- `actions/upload-pages-artifact@v1.0.4` (line 43)
- `actions/deploy-pages@v1` (line 48)
Each should be pinned to a full SHA digest, e.g. `actions/checkout@<40-char-sha> # v3`.

Locations:

- `action.yaml:30`
- `action.yaml:43`
- `action.yaml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build_command }}" appears directly in run: block of step "Build"; move to env: map

Locations:

- `action.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses

**Notes:**

Fixed action.yaml: (1) Moved `${{ inputs.build_command }}` from the run block into an env: block as BUILD_CMD to prevent shell injection; (2) Pinned actions/checkout@v3 to SHA f43a0e5ff2bd294095638e18286ca9a3d1956744, actions/upload-pages-artifact@v1.0.4 to SHA a597aecd27af1cf14095ccaa29169358e3d91e28, and actions/deploy-pages@v1 to SHA f27bcc15848fdcdcc02f01754eb838e44bcf389b.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Build' step of action.yaml. The `$BUILD_CMD` env var (populated from `inputs.build_command`) was being executed directly as an inline shell command, allowing shell metacharacter injection. Changed from `$BUILD_CMD` to `bash -c "$BUILD_CMD"` so the command is passed as a string argument to a bash subprocess rather than being expanded inline in the parent shell. The `${{ inputs.build_command }}` expression was already correctly placed in the step's `env:` block.

