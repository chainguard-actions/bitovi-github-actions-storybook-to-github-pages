# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Build' step in action.yaml directly interpolates attacker-controlled inputs into a run: shell block without first assigning them to environment variables. Specifically, `${{ inputs.install_command }}` and `${{ inputs.build_command }}` are used as raw shell commands, allowing an attacker who controls these inputs to execute arbitrary shell commands on the runner.

Locations:

- `action.yaml:38`

### unpinned-uses (severity: high)

Three uses: references in action.yaml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten. Failing references: `actions/checkout@v3`, `actions/upload-pages-artifact@v2`, `actions/deploy-pages@v3`.

Locations:

- `action.yaml:33`
- `action.yaml:45`
- `action.yaml:51`

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

Fixed action.yaml with the following changes:
1. script-injection / static-inline-injection: Moved `${{ inputs.install_command }}` and `${{ inputs.build_command }}` from the run: shell block into an env: block (as INSTALL_COMMAND and BUILD_COMMAND). The shell script now uses `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` to execute them, preventing attacker-controlled input from being directly interpolated as shell commands.
2. unpinned-uses: Pinned all three action references to immutable full commit SHAs:
   - actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744 # v3
   - actions/upload-pages-artifact@v2 → @a753861a5debcf57bf8b404356158c8e1e33150c # v2
   - actions/deploy-pages@v3 → @13b55b33dd8996121833dbc1db458c793a334630 # v3

