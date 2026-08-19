<!-- markdownlint-disable -->

# Hardening Report: typst-community--setup-typst/v4.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **typst-community--setup-typst/v4.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): `${{ github.ref_name }}` is interpolated directly inside a `run:` shell command string. An attacker who controls the branch/tag name (e.g. via a crafted release or dispatch) can inject arbitrary shell commands. Offending line: `run: git checkout -b ${{ github.ref_name }}-preview && git push -f origin ${{ github.ref_name }}-preview`. Fix by moving the value into an env var and quoting it: `env: REF_NAME: ${{ github.ref_name }}` then `run: git checkout -b "$REF_NAME"-preview && git push -f origin "$REF_NAME"-preview`.

Locations:

- `.github/workflows/publish.yml:37`

### unpinned-uses (severity: high)

Multiple `uses:` references in publish.yml are pinned to mutable tags instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if those tags are moved or compromised. Failing references: `actions/checkout@v4` (line 13), `actions/setup-node@v4` (line 14), `actions/github-script@v7` (line 29).

Locations:

- `.github/workflows/publish.yml:13`
- `.github/workflows/publish.yml:14`
- `.github/workflows/publish.yml:29`

### unpinned-uses (severity: high)

Multiple `uses:` references in test.yml are pinned to mutable tags instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks. Failing references: `actions/checkout@v4` (lines 33, 46, 63), `actions/setup-node@v4` (line 34), `actions/upload-artifact@v4` (line 38), `actions/download-artifact@v4` (lines 48, 65).

Locations:

- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:34`
- `.github/workflows/test.yml:38`
- `.github/workflows/test.yml:46`
- `.github/workflows/test.yml:48`
- `.github/workflows/test.yml:63`
- `.github/workflows/test.yml:65`

### missing-permissions (severity: medium)

test.yml has no top-level `permissions:` key and none of its jobs (`build`, `test-basic`, `test-zip-packages`) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A top-level `permissions: {}` or per-job minimal permissions should be added.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across publish.yml and test.yml:

1. script-injection (publish.yml): Moved `${{ github.ref_name }}` into an env var `REF_NAME` and referenced it as `"$REF_NAME"` in the shell command to prevent shell injection.

2. unpinned-uses (publish.yml): Pinned actions/checkout@v4→SHA 11d5960a, actions/setup-node@v4→SHA 49933ea5, actions/github-script@v7→SHA f28e40c7.

3. unpinned-uses (test.yml): Pinned actions/checkout@v4→SHA 11d5960a, actions/setup-node@v4→SHA 49933ea5, actions/upload-artifact@v4→SHA ea165f8d, actions/download-artifact@v4→SHA d3f86a10.

4. missing-permissions (test.yml): Added top-level `permissions: {}` and per-job `permissions: { contents: read }` for all three jobs (build, test-basic, test-zip-packages).

