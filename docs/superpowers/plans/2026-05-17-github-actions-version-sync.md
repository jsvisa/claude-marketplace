# GitHub Actions Version Sync Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add GitHub Actions workflows so that pushing a `v*` tag to any skill repo auto-syncs `package.json`, and the marketplace repo daily auto-syncs `marketplace.json` versions from each skill repo's latest tag.

**Architecture:** Four workflows total — one identical `sync-version.yml` per skill repo (tag push → update `package.json` → commit back to main), and one `sync-versions.yml` in the marketplace repo (daily cron → GitHub API → update `marketplace.json` plugin versions + patch-bump metadata version → commit).

**Tech Stack:** GitHub Actions, `actions/checkout@v4`, `jq`, `gh` CLI (pre-installed on `ubuntu-latest`), `GITHUB_TOKEN` (no extra secrets needed — public repo reads work with the built-in token).

---

### Task 1: Add sync-version workflow to debank-skill

**Files:**
- Create: `/Users/wenbiao.zheng/bc/debank-skill/.github/workflows/sync-version.yml`

- [ ] **Step 1: Create the workflow file**

```yaml
name: Sync package.json version

on:
  push:
    tags:
      - 'v*'

jobs:
  sync-version:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          ref: main

      - name: Extract version from tag
        id: version
        run: echo "version=${GITHUB_REF_NAME#v}" >> "$GITHUB_OUTPUT"

      - name: Update package.json
        run: |
          tmp=$(mktemp)
          jq --arg v "${{ steps.version.outputs.version }}" '.version = $v' package.json > "$tmp"
          mv "$tmp" package.json

      - name: Commit and push if changed
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          if git diff --quiet package.json; then
            echo "package.json already at ${GITHUB_REF_NAME#v}, skipping"
          else
            git add package.json
            git commit -m "chore: sync package.json to ${GITHUB_REF_NAME}"
            git push origin main
          fi
```

- [ ] **Step 2: Verify the file exists and is valid YAML**

```bash
cat /Users/wenbiao.zheng/bc/debank-skill/.github/workflows/sync-version.yml
```

Expected: full YAML printed with no errors.

- [ ] **Step 3: Commit and push**

```bash
cd /Users/wenbiao.zheng/bc/debank-skill
git add .github/workflows/sync-version.yml
git commit -m "ci: add workflow to sync package.json version on tag push"
git push
```

---

### Task 2: Add sync-version workflow to evm-decoder-skill

**Files:**
- Create: `/Users/wenbiao.zheng/bc/evm-decoder-skill/.github/workflows/sync-version.yml`

- [ ] **Step 1: Create the workflow file** (identical content to Task 1)

```yaml
name: Sync package.json version

on:
  push:
    tags:
      - 'v*'

jobs:
  sync-version:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          ref: main

      - name: Extract version from tag
        id: version
        run: echo "version=${GITHUB_REF_NAME#v}" >> "$GITHUB_OUTPUT"

      - name: Update package.json
        run: |
          tmp=$(mktemp)
          jq --arg v "${{ steps.version.outputs.version }}" '.version = $v' package.json > "$tmp"
          mv "$tmp" package.json

      - name: Commit and push if changed
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          if git diff --quiet package.json; then
            echo "package.json already at ${GITHUB_REF_NAME#v}, skipping"
          else
            git add package.json
            git commit -m "chore: sync package.json to ${GITHUB_REF_NAME}"
            git push origin main
          fi
```

- [ ] **Step 2: Verify the file exists and is valid YAML**

```bash
cat /Users/wenbiao.zheng/bc/evm-decoder-skill/.github/workflows/sync-version.yml
```

Expected: full YAML printed with no errors.

- [ ] **Step 3: Commit and push**

```bash
cd /Users/wenbiao.zheng/bc/evm-decoder-skill
git add .github/workflows/sync-version.yml
git commit -m "ci: add workflow to sync package.json version on tag push"
git push
```

---

### Task 3: Add sync-version workflow to solexplain

**Files:**
- Create: `/Users/wenbiao.zheng/bc/solexplain/.github/workflows/sync-version.yml`

Note: solexplain already has `.github/workflows/ci.yml` — this is a separate file, keep them independent.

- [ ] **Step 1: Create the workflow file** (identical content to Tasks 1 and 2)

```yaml
name: Sync package.json version

on:
  push:
    tags:
      - 'v*'

jobs:
  sync-version:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          ref: main

      - name: Extract version from tag
        id: version
        run: echo "version=${GITHUB_REF_NAME#v}" >> "$GITHUB_OUTPUT"

      - name: Update package.json
        run: |
          tmp=$(mktemp)
          jq --arg v "${{ steps.version.outputs.version }}" '.version = $v' package.json > "$tmp"
          mv "$tmp" package.json

      - name: Commit and push if changed
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          if git diff --quiet package.json; then
            echo "package.json already at ${GITHUB_REF_NAME#v}, skipping"
          else
            git add package.json
            git commit -m "chore: sync package.json to ${GITHUB_REF_NAME}"
            git push origin main
          fi
```

- [ ] **Step 2: Verify the file exists and is valid YAML**

```bash
cat /Users/wenbiao.zheng/bc/solexplain/.github/workflows/sync-version.yml
```

Expected: full YAML printed with no errors.

- [ ] **Step 3: Commit and push**

```bash
cd /Users/wenbiao.zheng/bc/solexplain
git add .github/workflows/sync-version.yml
git commit -m "ci: add workflow to sync package.json version on tag push"
git push
```

---

### Task 4: Add sync-versions workflow to claude-marketplace

**Files:**
- Create: `/Users/wenbiao.zheng/bc/claude-marketplace/.github/workflows/sync-versions.yml`

- [ ] **Step 1: Create the workflow file**

```yaml
name: Sync marketplace plugin versions

on:
  schedule:
    - cron: '7 9 * * *'  # daily at 09:07 UTC
  workflow_dispatch:

jobs:
  sync-versions:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@v4

      - name: Sync plugin versions and bump marketplace metadata
        run: |
          set -e
          MARKETPLACE_FILE=".claude-plugin/marketplace.json"
          CHANGED=false

          while IFS=$'\t' read -r plugin_name plugin_url current_version; do
            repo=$(echo "$plugin_url" | sed 's|https://github.com/||; s|\.git$||')
            latest=$(gh api "repos/${repo}/tags" --jq 'if length > 0 then first | .name | ltrimstr("v") else "" end')

            if [ -n "$latest" ] && [ "$latest" != "$current_version" ]; then
              echo "Updating $plugin_name: $current_version -> $latest"
              tmp=$(mktemp)
              jq --arg name "$plugin_name" --arg ver "$latest" \
                '(.plugins[] | select(.name == $name) | .version) = $ver' \
                "$MARKETPLACE_FILE" > "$tmp" && mv "$tmp" "$MARKETPLACE_FILE"
              CHANGED=true
            else
              echo "$plugin_name is up to date ($current_version)"
            fi
          done < <(jq -r '.plugins[] | [.name, .source.url, .version] | @tsv' "$MARKETPLACE_FILE")

          if [ "$CHANGED" = "true" ]; then
            current_meta=$(jq -r '.metadata.version' "$MARKETPLACE_FILE")
            major=$(echo "$current_meta" | cut -d. -f1)
            minor=$(echo "$current_meta" | cut -d. -f2)
            patch=$(echo "$current_meta" | cut -d. -f3)
            new_meta="${major}.${minor}.$((patch + 1))"

            tmp=$(mktemp)
            jq --arg v "$new_meta" '.metadata.version = $v' "$MARKETPLACE_FILE" > "$tmp" && mv "$tmp" "$MARKETPLACE_FILE"

            git config user.name "github-actions[bot]"
            git config user.email "github-actions[bot]@users.noreply.github.com"
            git add "$MARKETPLACE_FILE"
            git commit -m "chore: sync plugin versions (marketplace ${new_meta})"
            git push
          else
            echo "All plugin versions are up to date, nothing to commit."
          fi
```

- [ ] **Step 2: Verify the file exists and is valid YAML**

```bash
cat /Users/wenbiao.zheng/bc/claude-marketplace/.github/workflows/sync-versions.yml
```

Expected: full YAML printed with no errors.

- [ ] **Step 3: Commit and push**

```bash
cd /Users/wenbiao.zheng/bc/claude-marketplace
git add .github/workflows/sync-versions.yml
git commit -m "ci: add daily workflow to sync plugin versions in marketplace.json"
git push
```

---

### Task 5: Verify the marketplace workflow runs correctly

- [ ] **Step 1: Trigger a manual run**

```bash
cd /Users/wenbiao.zheng/bc/claude-marketplace
gh workflow run sync-versions.yml
```

Expected output: `Created workflow_dispatch event for sync-versions.yml at main`

- [ ] **Step 2: Watch the run until it completes**

```bash
gh run watch $(gh run list --workflow=sync-versions.yml --limit=1 --json databaseId --jq '.[0].databaseId')
```

Expected: run completes with green checkmark (✓ sync-versions)

- [ ] **Step 3: Check the run log to confirm correct behavior**

```bash
gh run view $(gh run list --workflow=sync-versions.yml --limit=1 --json databaseId --jq '.[0].databaseId') --log
```

Expected: log shows each plugin name with either `is up to date` or `Updating X: old -> new`, followed by either a commit or `All plugin versions are up to date, nothing to commit.`
