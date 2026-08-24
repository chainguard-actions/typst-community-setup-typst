<!-- markdownlint-disable -->

# Hardening Report: typst-community--setup-typst/v5.3.0b1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **typst-community--setup-typst/v5.3.0b1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags instead of immutable 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if those tags are moved.

publish.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `actions/github-script@v8`
test.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `actions/upload-artifact@v6`, `actions/download-artifact@v8`

Locations:

- `.github/workflows/publish.yml:11`
- `.github/workflows/publish.yml:12`
- `.github/workflows/publish.yml:29`
- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:29`
- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:42`
- `.github/workflows/test.yml:43`

### script-injection (severity: high)

Rule (a) violation: `publish.yml` contains a `run:` block that directly interpolates `${{ github.ref_name }}` into the shell command string. This allows an attacker who can control the ref name (e.g. via a specially crafted branch or tag name) to inject arbitrary shell commands.

Offending line: `run: git checkout -b ${{ github.ref_name }}-preview && git push -f origin ${{ github.ref_name }}-preview`

Fix: move the value into an `env:` variable and double-quote it in the shell script, e.g.:
```yaml
env:
  REF_NAME: ${{ github.ref_name }}
run: git checkout -b "$REF_NAME"-preview && git push -f origin "$REF_NAME"-preview
```

Locations:

- `.github/workflows/publish.yml:35`

### missing-permissions (severity: medium)

`test.yml` has no top-level `permissions:` key and none of its jobs (`build`, `test-basic`, `test-executable-name`, `test-typst-versions`, `test-zip-packages`) define a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal explicit `permissions:` block (e.g. `contents: read`) should be added at the top level or on each job.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned all action references in publish.yml and test.yml to full 40-char commit SHAs (actions/checkout@d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-node@249970729cb0ef3589644e2896645e5dc5ba9c38, actions/github-script@ed597411d8f924073f98dfc5c65a23a2325f34cd, actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f, actions/download-artifact@3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c), preserving the tag as a comment.
2. script-injection: In publish.yml, moved ${{ github.ref_name }} out of the run: shell string into an env: block as REF_NAME, and referenced it as "$REF_NAME" (double-quoted) in the shell script.
3. missing-permissions: Added top-level 'permissions: contents: read' block to test.yml to enforce least-privilege access.

