<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-download-artifact/v21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-download-artifact/v21** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned full-length SHA digests, making them vulnerable to supply-chain attacks.

- `.github/workflows/download.yml`: `actions/checkout@v6` (used 17 times across all jobs)
- `.github/workflows/upload.yml`: `actions/upload-artifact@v7` (used 3 times)
- `.github/workflows/npm-updates.yml`: `dawidd6/reusable-workflows/.github/workflows/npm-updates.yml@master` (branch ref)

Locations:

- `.github/workflows/download.yml:21`
- `.github/workflows/upload.yml:18`
- `.github/workflows/npm-updates.yml:6`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job defines its own `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All three files are affected: download.yml, upload.yml, and npm-updates.yml.

Locations:

- `.github/workflows/download.yml:1`
- `.github/workflows/upload.yml:1`
- `.github/workflows/npm-updates.yml:1`

### script-injection (severity: high)

Two `run:` steps in download.yml directly interpolate a `${{ ... }}` expression inside a shell command string (sub-rule a). Even though `steps.download.outputs.dry_run` appears internal, it flows through YAML template substitution before the shell processes it, enabling script injection if the value contains shell metacharacters.

Offending lines:
- `run: test ${{ steps.download.outputs.dry_run }} == true` (job: download-dry-run-exists)
- `run: test ${{ steps.download.outputs.dry_run }} == false` (job: download-dry-run-not-exists)

Fix: use an env var instead — e.g. `env: DRY_RUN: ${{ steps.download.outputs.dry_run }}` and then `run: test "$DRY_RUN" == true`.

Locations:

- `.github/workflows/download.yml:196`
- `.github/workflows/download.yml:214`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across the three workflow files:

1. unpinned-uses: Pinned actions/checkout@v6 to full SHA d23441a48e516b6c34aea4fa41551a30e30af803 (17 occurrences in download.yml), actions/upload-artifact@v7 to 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a (3 occurrences in upload.yml), and dawidd6/reusable-workflows branch ref 'master' to eef24d408f08a926601a42fd4051807bcf3d3569 in npm-updates.yml.

2. missing-permissions: Added 'permissions: {}' top-level block to all three workflow files.

3. script-injection: Fixed both dry_run test steps in download.yml by moving ${{ steps.download.outputs.dry_run }} into an env: block as DRY_RUN and referencing it as "$DRY_RUN" in the shell command.

