<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-download-artifact/v19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-download-artifact/v19** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks:
- download.yml: `uses: actions/checkout@v6` (tag, appears in every job)
- npm-updates.yml: `uses: dawidd6/reusable-workflows/.github/workflows/npm-updates.yml@master` (branch ref)
- upload.yml: `uses: actions/upload-artifact@v7` (tag)

Locations:

- `.github/workflows/download.yml:23`
- `.github/workflows/npm-updates.yml:6`
- `.github/workflows/upload.yml:20`

### script-injection (severity: high)

Two `run:` steps in download.yml directly interpolate a GitHub Actions expression into a shell command string (sub-rule a). `${{ steps.download.outputs.dry_run }}` is expanded by the Actions template engine before the shell sees it, allowing an attacker who controls the step output to inject arbitrary shell commands.

Offending lines:
  `run: test ${{ steps.download.outputs.dry_run }} == true`
  `run: test ${{ steps.download.outputs.dry_run }} == false`

Fix: capture the value in an env var and quote it: `env: DRY_RUN: ${{ steps.download.outputs.dry_run }}` then `run: test "$DRY_RUN" == true`

Locations:

- `.github/workflows/download.yml:148`
- `.github/workflows/download.yml:165`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` block, and no individual job within them declares job-level permissions. Without explicit permissions, workflows run with the default repository permissions (which may be read/write for contents and other scopes), violating the principle of least privilege.

Locations:

- `.github/workflows/download.yml:1`
- `.github/workflows/npm-updates.yml:1`
- `.github/workflows/upload.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across the three workflow files:

1. unpinned-uses: Pinned all mutable references to full commit SHAs with tag comments:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 # v6 (in download.yml, all 16 occurrences)
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7 (in upload.yml, 3 occurrences)
   - dawidd6/reusable-workflows/.github/workflows/npm-updates.yml@master → @eef24d408f08a926601a42fd4051807bcf3d3569 # master (in npm-updates.yml)

2. script-injection: Fixed both offending steps in download.yml by moving `${{ steps.download.outputs.dry_run }}` into an `env:` block as `DRY_RUN` and using `test "$DRY_RUN" == true/false` in the shell command.

3. missing-permissions: Added `permissions: {}` top-level block to all three workflow files (download.yml, npm-updates.yml, upload.yml).

