<!-- markdownlint-disable -->

# Hardening Report: typst-community--setup-typst/v5.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **typst-community--setup-typst/v5.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command string. The step `run: git checkout -b ${{ github.ref_name }}-preview && git push -f origin ${{ github.ref_name }}-preview` injects `github.ref_name` directly into the shell command. An attacker who can control the ref name (e.g. via a specially crafted branch name) could inject arbitrary shell commands. The value should be passed via an env: variable and double-quoted in the script instead.

Locations:

- `.github/workflows/publish.yml:38`

### unpinned-uses (severity: high)

Multiple `uses:` references in publish.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if those tags are moved: `actions/checkout@v6` (line 13), `actions/setup-node@v6` (line 14), `actions/github-script@v8` (line 29).

Locations:

- `.github/workflows/publish.yml:13`
- `.github/workflows/publish.yml:14`
- `.github/workflows/publish.yml:29`

### unpinned-uses (severity: high)

Multiple `uses:` references in test.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks: `actions/checkout@v6` (lines 29, 43, 57, 79, 109), `actions/setup-node@v6` (line 30), `actions/upload-artifact@v6` (line 35), `actions/download-artifact@v8` (lines 44, 58, 80, 110).

Locations:

- `.github/workflows/test.yml:29`
- `.github/workflows/test.yml:30`
- `.github/workflows/test.yml:35`
- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:44`
- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:58`
- `.github/workflows/test.yml:79`
- `.github/workflows/test.yml:80`
- `.github/workflows/test.yml:109`
- `.github/workflows/test.yml:110`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and none of its jobs (build, test-basic, test-executable-name, test-typst-versions, test-zip-packages) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. write access to contents). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across two workflow files:

1. publish.yml - script-injection: Moved `${{ github.ref_name }}` out of the `run:` shell command into an `env:` block as `REF_NAME`, then referenced it as `"$REF_NAME-preview"` (double-quoted) in the shell script.

2. publish.yml - unpinned-uses: Pinned all 3 action references to full 40-char commit SHAs: actions/checkout@v6 → df4cb1c..., actions/setup-node@v6 → 249970729..., actions/github-script@v8 → ed597411...

3. test.yml - unpinned-uses: Pinned all action references to full SHAs: actions/checkout@v6 (5 occurrences) → df4cb1c..., actions/setup-node@v6 → 249970729..., actions/upload-artifact@v6 → b7c566a..., actions/download-artifact@v8 (4 occurrences) → 3e5f45b...

4. test.yml - missing-permissions: Added top-level `permissions: contents: read` block, providing the minimum access needed for checkout operations while restricting all other permissions.

