# GitHub Actions Version Sync Design

**Date:** 2026-05-17

## Problem

Version numbers across the ecosystem are maintained in three places:

1. Git tags on each skill repo (e.g. `v1.1.0`) — source of truth
2. `package.json` in each skill repo — often left stale after tagging
3. `marketplace.json` in `claude-marketplace` — updated manually, easy to forget

Goal: automate both sync points so no manual version bookkeeping is needed.

## Scope

Four repos:
- `jsvisa/debank-skill`
- `jsvisa/evm-decoder-skill`
- `jsvisa/solexplain`
- `jsvisa/claude-marketplace`

## Design

### Skill Repos (debank-skill, evm-decoder-skill, solexplain)

**File:** `.github/workflows/sync-version.yml`

**Trigger:** `push: tags: ['v*']`

**Steps:**
1. Checkout repo with default `GITHUB_TOKEN` (write permission for push)
2. Extract version by stripping `v` prefix from `GITHUB_REF_NAME`
3. Update `package.json` `version` field using `jq`
4. If `package.json` changed, commit `chore: sync package.json to vX.Y.Z` and push to `main`

**Notes:**
- Uses only built-in `GITHUB_TOKEN` — no extra secrets required
- solexplain already has `ci.yml`; the new workflow is a separate file to keep concerns separate
- No-op if `package.json` version already matches the tag

### claude-marketplace

**File:** `.github/workflows/sync-versions.yml`

**Trigger:**
- `schedule: '7 9 * * *'` (daily at 09:07 UTC, off-minute to avoid fleet pileup)
- `workflow_dispatch` for manual runs

**Steps:**
1. Checkout with `GITHUB_TOKEN`
2. For each plugin in `marketplace.json`, call `gh api repos/jsvisa/{repo}/git/refs/tags` and extract the latest `v*` tag
3. Compare latest tag version against current version in `marketplace.json`
4. If any plugin version changed: update the plugin version in `marketplace.json` using `jq`, then patch-bump `metadata.version` (e.g. `1.4.0` → `1.4.1`)
5. Commit `chore: sync plugin versions` and push to `main`
6. If nothing changed: skip commit (no-op run)

**Notes:**
- Uses only `GITHUB_TOKEN` — reads other repos via public GitHub API, no cross-repo secrets
- Marketplace `metadata.version` is patch-bumped on every sync that changes something
- Plugin repo list is derived from existing `marketplace.json` `plugins[].source.url` entries — no hardcoding

## What This Replaces

All manual steps:
- ~~Bump `package.json` after tagging a skill~~
- ~~Update `marketplace.json` after publishing a skill~~
- ~~Push marketplace changes~~
