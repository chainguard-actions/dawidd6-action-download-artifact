<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-download-artifact/v23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-download-artifact/v23** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned full-length SHA digests, making them vulnerable to supply-chain attacks.

- .github/workflows/download.yml: `actions/checkout@v7` (used 18 times across all jobs)
- .github/workflows/upload.yml: `actions/upload-artifact@v7` (used 4 times)
- .github/workflows/npm-updates.yml: `dawidd6/reusable-workflows/.github/workflows/npm-updates.yml@master` (branch ref)

Locations:

- `.github/workflows/download.yml:22`
- `.github/workflows/upload.yml:18`
- `.github/workflows/npm-updates.yml:8`

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in download.yml directly interpolate a GitHub Actions expression `${{ steps.download.outputs.found_artifact }}` inside a shell command string. Even though `steps.*.outputs.*` may appear controlled, any `${{ ... }}` expression interpolated directly into a `run:` block is evaluated before the shell sees it, allowing injection of shell metacharacters if the value is attacker-influenced.

Offending lines:
- `run: test ${{ steps.download.outputs.found_artifact }} == true`
- `run: test ${{ steps.download.outputs.found_artifact }} == false`

Locations:

- `.github/workflows/download.yml:193`
- `.github/workflows/download.yml:213`

### missing-permissions (severity: medium)

The workflow file npm-updates.yml has no top-level `permissions:` key and the single job (which calls a reusable workflow via `uses:`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/npm-updates.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings: (1) Pinned actions/checkout@v7 → SHA 3d3c42e5... (18 occurrences in download.yml), actions/upload-artifact@v7 → SHA 043fb46d... (4 occurrences in upload.yml), and dawidd6/reusable-workflows@master → SHA eef24d40... in npm-updates.yml. (2) Fixed script injection in download-dry-run-exists and download-dry-run-not-exists jobs by moving ${{ steps.download.outputs.found_artifact }} into env: blocks and referencing as $FOUND_ARTIFACT. (3) Added `permissions: {}` to npm-updates.yml.

