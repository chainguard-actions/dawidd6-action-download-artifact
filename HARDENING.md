<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-download-artifact/v22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-download-artifact/v22** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` steps in download.yml directly interpolate `${{ steps.download.outputs.dry_run }}` (a `steps.*.outputs.*` expression) inside shell commands without routing through an env var. This is a script-injection violation (rule a): any `${{ ... }}` expression inside a `run:` block is unsafe because YAML template substitution occurs before the shell ever sees the value.

Offending lines:
- `run: test ${{ steps.download.outputs.dry_run }} == true` (download-dry-run-exists job)
- `run: test ${{ steps.download.outputs.dry_run }} == false` (download-dry-run-not-exists job)

Fix: move the value into an env var and double-quote it in the shell, e.g.:
```yaml
env:
  DRY_RUN: ${{ steps.download.outputs.dry_run }}
run: test "$DRY_RUN" == true
```

Locations:

- `.github/workflows/download.yml:168`
- `.github/workflows/download.yml:181`

### permissions (severity: medium)

Workflow file download.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/download.yml:1`

### permissions (severity: medium)

Workflow file npm-updates.yml has no top-level `permissions:` key and no job-level `permissions:` key on its job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions.

Locations:

- `.github/workflows/npm-updates.yml:1`

### permissions (severity: medium)

Workflow file upload.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions.

Locations:

- `.github/workflows/upload.yml:1`

### unpinned-uses (severity: high)

Multiple `uses:` references in download.yml are pinned to mutable version tags (`@v7`) rather than immutable 40-character commit SHAs. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks.

All 17 occurrences of `uses: actions/checkout@v7` are unpinned.

Fix example: `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`

Locations:

- `.github/workflows/download.yml:23`

### unpinned-uses (severity: high)

Multiple `uses:` references in upload.yml are pinned to mutable version tags (`@v7`) rather than immutable 40-character commit SHAs.

All 3 occurrences of `uses: actions/upload-artifact@v7` are unpinned.

Fix example: `uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4`

Locations:

- `.github/workflows/upload.yml:20`

### unpinned-uses (severity: high)

The `uses:` reference in npm-updates.yml is pinned to a mutable branch name (`@master`) rather than an immutable 40-character commit SHA. Branch references are especially dangerous as they track the latest commit on that branch.

Offending line: `uses: dawidd6/reusable-workflows/.github/workflows/npm-updates.yml@master`

Locations:

- `.github/workflows/npm-updates.yml:5`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, permissions, unpinned-uses

**Notes:**

Fixed all 7 findings across 3 workflow files:
1. download.yml: Added `permissions: {}` top-level block; pinned all 17 `actions/checkout@v7` references to SHA `3d3c42e5aac5ba805825da76410c181273ba90b1 # v7`; fixed script injection in `download-dry-run-exists` and `download-dry-run-not-exists` jobs by moving `${{ steps.download.outputs.dry_run }}` into an `env: DRY_RUN:` block and referencing `"$DRY_RUN"` in the shell.
2. npm-updates.yml: Added `permissions: {}` top-level block; pinned `dawidd6/reusable-workflows@master` to SHA `eef24d408f08a926601a42fd4051807bcf3d3569 # master`.
3. upload.yml: Added `permissions: {}` top-level block; pinned all 3 `actions/upload-artifact@v7` references to SHA `043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7`.

