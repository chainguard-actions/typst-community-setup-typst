<!-- markdownlint-disable -->

# Hardening Report: typst-community--setup-typst/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **typst-community--setup-typst/v5.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` step in publish.yml directly interpolates `${{ github.ref_name }}` into a shell command string: `run: git checkout -b ${{ github.ref_name }}-preview && git push -f origin ${{ github.ref_name }}-preview`. The `github.ref_name` value is attacker-influenced (e.g., via crafted branch or tag names) and is substituted by the YAML template engine before the shell ever sees it, enabling command injection. It must be passed via an `env:` variable and double-quoted in the shell instead.

Locations:

- `.github/workflows/publish.yml:36`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced tags are moved or compromised. Failing references in publish.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `actions/github-script@v8`. Failing references in test.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `actions/upload-artifact@v6`, `actions/download-artifact@v8`.

Locations:

- `.github/workflows/publish.yml:10`
- `.github/workflows/publish.yml:11`
- `.github/workflows/publish.yml:28`
- `.github/workflows/test.yml:26`
- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:32`
- `.github/workflows/test.yml:41`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and none of its jobs (`build`, `test-basic`, `test-executable-name`, `test-versions-map`, `test-zip-packages`) define a `permissions:` block. This means the workflow runs with GitHub's default permissions, which may be broader than necessary (e.g., `contents: write` by default on some repository configurations). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings:

1. script-injection (publish.yml line 36): Moved `${{ github.ref_name }}` out of the `run:` shell string into an `env:` block as `REF_NAME`. The shell command now uses double-quoted `"$REF_NAME-preview"` to prevent command injection via crafted branch/tag names.

2. unpinned-uses: Pinned all five action references to full 40-character SHA hashes (resolved via lookup_action_sha), preserving the original tag as a comment:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - actions/github-script@v8 → @ed597411d8f924073f98dfc5c65a23a2325f34cd
   - actions/upload-artifact@v6 → @b7c566a772e6b6bfb58ed0dc250532a479d7789f
   - actions/download-artifact@v8 → @3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c

3. missing-permissions (test.yml): Added `permissions: {}` at the top level of test.yml, granting no permissions since the test workflow only reads code and runs tests without needing any GitHub API access.

