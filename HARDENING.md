<!-- markdownlint-disable -->

# Hardening Report: typst-community--setup-typst/v5.3.0b2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **typst-community--setup-typst/v5.3.0b2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: `${{ github.ref_name }}` is directly interpolated inside a `run:` shell command string in publish.yml. An attacker who can influence the ref name (e.g. via a specially crafted branch name) could inject arbitrary shell commands. The offending line is: `run: git checkout -b ${{ github.ref_name }}-preview && git push -f origin ${{ github.ref_name }}-preview`. Fix by routing through an env var and double-quoting: `env: REF_NAME: ${{ github.ref_name }}` then `run: git checkout -b "$REF_NAME"-preview && git push -f origin "$REF_NAME"-preview`.

Locations:

- `.github/workflows/publish.yml:38`

### unpinned-uses (severity: high)

Multiple `uses:` references in publish.yml are pinned to mutable tags rather than full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if those tags are moved. Failing references: `actions/checkout@v7` (line 13), `actions/setup-node@v7` (line 14), `actions/github-script@v9` (line 29). All should be pinned to a full SHA, e.g. `actions/checkout@<40-hex-sha> # v7`.

Locations:

- `.github/workflows/publish.yml:13`
- `.github/workflows/publish.yml:14`
- `.github/workflows/publish.yml:29`

### unpinned-uses (severity: high)

Multiple `uses:` references in test.yml are pinned to mutable tags rather than full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks. Failing references include: `actions/checkout@v7`, `actions/setup-node@v7`, `actions/upload-artifact@v7`, `actions/download-artifact@v8` (appearing in multiple jobs). All should be pinned to a full SHA, e.g. `actions/checkout@<40-hex-sha> # v7`.

Locations:

- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:21`
- `.github/workflows/test.yml:26`
- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:34`
- `.github/workflows/test.yml:44`
- `.github/workflows/test.yml:45`
- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:58`
- `.github/workflows/test.yml:79`
- `.github/workflows/test.yml:80`

### missing-permissions (severity: medium)

test.yml has no top-level `permissions:` key and none of its jobs (build, test-basic, test-executable-name, test-typst-versions, test-zip-packages) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Add a top-level `permissions: {}` (or minimal specific scopes) to restrict the GITHUB_TOKEN.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings:
1. script-injection (publish.yml line 38): Moved `${{ github.ref_name }}` into an `env:` block as `REF_NAME` and updated the run command to use `"$REF_NAME"` with double-quoting.
2. unpinned-uses (publish.yml): Pinned actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1, actions/setup-node@v7 → @820762786026740c76f36085b0efc47a31fe5020, actions/github-script@v9 → @3a2844b7e9c422d3c10d287c895573f7108da1b3.
3. unpinned-uses (test.yml): Pinned all occurrences of actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1, actions/setup-node@v7 → @820762786026740c76f36085b0efc47a31fe5020, actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a, actions/download-artifact@v8 → @3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c.
4. missing-permissions (test.yml): Added `permissions: {}` at the top level to restrict GITHUB_TOKEN to no permissions by default.

