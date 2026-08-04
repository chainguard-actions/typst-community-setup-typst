<!-- markdownlint-disable -->

# Hardening Report: typst-community--setup-typst/v5.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **typst-community--setup-typst/v5.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable version tags (e.g. @v6, @v8) rather than immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the upstream action tag is moved or compromised.

In publish.yml: actions/checkout@v6, actions/setup-node@v6, actions/github-script@v8
In test.yml: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v6, actions/download-artifact@v8

Locations:

- `.github/workflows/publish.yml:11`
- `.github/workflows/publish.yml:12`
- `.github/workflows/publish.yml:28`
- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:32`
- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:47`

### script-injection (severity: high)

Sub-rule (a): A `run:` step in publish.yml directly interpolates `${{ github.ref_name }}` into a shell command string. Before the shell executes the command, GitHub Actions substitutes the expression value verbatim, allowing an attacker who controls the ref name (e.g. via a specially crafted branch name) to inject arbitrary shell commands.

Offending line: `run: git checkout -b ${{ github.ref_name }}-preview && git push -f origin ${{ github.ref_name }}-preview`

Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g.:
```yaml
env:
  REF_NAME: ${{ github.ref_name }}
run: git checkout -b "$REF_NAME-preview" && git push -f origin "$REF_NAME-preview"
```

Locations:

- `.github/workflows/publish.yml:38`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key, and none of its jobs (build, test-basic, test-executable-name, test-typst-versions, test-zip-packages) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. write access to contents). A minimal explicit permissions block such as `permissions: read-all` or specific scopes should be added.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned all action references in publish.yml and test.yml to full 40-char commit SHAs (actions/checkout@d23441a4, actions/setup-node@249970729, actions/github-script@ed597411, actions/upload-artifact@b7c566a7, actions/download-artifact@3e5f45b2), preserving the original tag in a comment.
2. script-injection: In publish.yml, moved `${{ github.ref_name }}` out of the run: shell string into an env: block as REF_NAME, and updated the shell command to use quoted "$REF_NAME" variable.
3. missing-permissions: Added top-level `permissions: contents: read` to test.yml to enforce least-privilege access (the workflow only needs to read repository contents).

