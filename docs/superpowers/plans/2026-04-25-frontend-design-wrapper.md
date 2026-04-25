# Frontend Design Wrapper Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert this template plugin repository into the Accelerate Data wrapper for Anthropic's official `frontend-design` skill, with Claude and Codex plugin manifests, reviewed sparse upstream sync, and no local promptfoo eval harness.

**Architecture:** The repository remains a normal whole-repo plugin source whose runtime content lives under `skills/frontend-design/`. Upstream-owned files are copied into the wrapper by a GitHub Actions workflow that opens a pull request, while wrapper-owned manifests, guidance, docs, and workflow files stay local and reviewable.

**Tech Stack:** Markdown, JSON plugin manifests, GitHub Actions, POSIX shell validation, sparse Git checkout.

---

## File Structure

- Modify `.claude-plugin/plugin.json` to identify the wrapper plugin as `frontend-design` for Claude.
- Modify `.codex-plugin/plugin.json` to identify the wrapper plugin as `frontend-design` for Codex.
- Delete `skills/example-skill/SKILL.md` because the template skill is replaced by the official upstream skill.
- Create `skills/frontend-design/SKILL.md` by copying `plugins/frontend-design/skills/frontend-design/SKILL.md` from `anthropics/claude-plugins-official`.
- Replace `LICENSE` with the upstream `plugins/frontend-design/LICENSE` content copied during sync.
- Modify `README.md` to describe this wrapper repository, upstream attribution, local usage, sync behavior, and marketplace source shape.
- Modify `AGENTS.md` to replace template guidance with wrapper-specific durability rules, upstream-owned content boundaries, sync warning rules, and validation expectations.
- Modify `CLAUDE.md` only if needed to keep it as a lightweight pointer to `AGENTS.md`.
- Modify `repo-map.json` to remove template eval harness commands and describe the wrapper repo, sync workflow, and validation commands.
- Create `.github/workflows/sync-upstream.yml` to sparse-checkout the official upstream plugin, copy approved files, validate wrapper integrity, and open a review PR.
- Keep `.github/workflows/version-bump-check.yml` unless the implementation deliberately removes versioning from the Claude manifest; if kept, bump `.claude-plugin/plugin.json.version`.
- Keep `.githooks/pre-commit` as-is unless validation needs to be added later.
- Delete `tests/evals/` entirely because this wrapper packages upstream-owned behavior and does not maintain local promptfoo coverage.

---

### Task 1: Capture Current State And Upstream Inventory

**Files:**
- Read: `docs/superpowers/specs/2026-04-25-frontend-design-wrapper-design.md`
- Read: `repo-map.json`
- Read: `.claude-plugin/plugin.json`
- Read: `.codex-plugin/plugin.json`
- Read: `README.md`
- Read: `AGENTS.md`
- Inspect upstream: `https://github.com/anthropics/claude-plugins-official`, path `plugins/frontend-design/`

- [ ] **Step 1: Confirm the worktree state**

Run:

```bash
git status --short
```

Expected: only planned local docs may be untracked or modified before implementation starts. Do not revert user-authored files.

- [ ] **Step 2: Confirm the repo is still the template wrapper**

Run:

```bash
rg --files
```

Expected output includes:

```text
skills/example-skill/SKILL.md
tests/evals/package.json
.claude-plugin/plugin.json
.codex-plugin/plugin.json
```

- [ ] **Step 3: Confirm upstream frontend-design file inventory**

Run:

```bash
tmpdir="$(mktemp -d)"
git clone --depth 1 --filter=blob:none --sparse \
  https://github.com/anthropics/claude-plugins-official.git \
  "$tmpdir/claude-plugins-official"
cd "$tmpdir/claude-plugins-official"
git sparse-checkout set plugins/frontend-design
find plugins/frontend-design -maxdepth 4 -type f | sort
cd -
rm -rf "$tmpdir"
```

Expected output includes exactly these relevant files:

```text
plugins/frontend-design/.claude-plugin/plugin.json
plugins/frontend-design/LICENSE
plugins/frontend-design/README.md
plugins/frontend-design/skills/frontend-design/SKILL.md
```

- [ ] **Step 4: Commit is not required for inventory**

Do not commit after this task; it only establishes facts for the following edits.

---

### Task 2: Replace Template Runtime Content With Upstream Skill

**Files:**
- Delete: `skills/example-skill/SKILL.md`
- Create: `skills/frontend-design/SKILL.md`
- Modify: `LICENSE`

- [ ] **Step 1: Remove the template skill directory**

Run:

```bash
rm -rf skills/example-skill
mkdir -p skills/frontend-design
```

Expected: `skills/example-skill/` no longer exists and `skills/frontend-design/` exists.

- [ ] **Step 2: Copy upstream runtime files**

Run from repo root:

```bash
tmpdir="$(mktemp -d)"
git clone --depth 1 --filter=blob:none --sparse \
  https://github.com/anthropics/claude-plugins-official.git \
  "$tmpdir/claude-plugins-official"
cd "$tmpdir/claude-plugins-official"
git sparse-checkout set plugins/frontend-design
cd -
cp "$tmpdir/claude-plugins-official/plugins/frontend-design/skills/frontend-design/SKILL.md" \
  skills/frontend-design/SKILL.md
cp "$tmpdir/claude-plugins-official/plugins/frontend-design/LICENSE" LICENSE
rm -rf "$tmpdir"
```

Expected:

```bash
test -f skills/frontend-design/SKILL.md
test -f LICENSE
test ! -e skills/example-skill
```

- [ ] **Step 3: Verify copied skill has no local wrapper edits**

Run:

```bash
sed -n '1,80p' skills/frontend-design/SKILL.md
rg -n "allowed-tools|slash command|\\.claude/|\\.codex/" skills/frontend-design/SKILL.md || true
```

Expected: the first command shows the official frontend design guidance, and the second command produces no matches.

- [ ] **Step 4: Commit runtime replacement**

Run:

```bash
git add skills LICENSE
git commit -m "chore: add upstream frontend-design skill"
```

Expected: commit succeeds with the template skill removed and upstream skill files added.

---

### Task 3: Update Plugin Manifests

**Files:**
- Modify: `.claude-plugin/plugin.json`
- Modify: `.codex-plugin/plugin.json`

- [ ] **Step 1: Update the Claude manifest**

Replace `.claude-plugin/plugin.json` with:

```json
{
  "name": "frontend-design",
  "description": "Frontend design skill for production UI and UX implementation.",
  "version": "0.1.1",
  "author": {
    "name": "Accelerate Data"
  },
  "repository": "https://github.com/accelerate-data/frontend-design",
  "license": "See LICENSE",
  "keywords": [
    "skills",
    "claude-code",
    "frontend",
    "design",
    "ui",
    "ux"
  ],
  "skills": "./skills"
}
```

- [ ] **Step 2: Update the Codex manifest**

Replace `.codex-plugin/plugin.json` with:

```json
{
  "name": "frontend-design",
  "description": "Frontend design skill for production UI and UX implementation.",
  "author": {
    "name": "Accelerate Data"
  },
  "repository": "https://github.com/accelerate-data/frontend-design",
  "license": "See LICENSE",
  "keywords": [
    "skills",
    "codex",
    "claude-code",
    "frontend",
    "design",
    "ui",
    "ux"
  ],
  "skills": "./skills/",
  "interface": {
    "displayName": "Frontend Design",
    "shortDescription": "Production frontend design guidance for UI and UX implementation.",
    "developerName": "Accelerate Data",
    "category": "Coding",
    "capabilities": [],
    "screenshots": []
  }
}
```

- [ ] **Step 3: Validate manifest JSON**

Run:

```bash
jq empty .claude-plugin/plugin.json
jq empty .codex-plugin/plugin.json
```

Expected: both commands exit 0 with no output.

- [ ] **Step 4: Commit manifest changes**

Run:

```bash
git add .claude-plugin/plugin.json .codex-plugin/plugin.json
git commit -m "chore: update frontend-design manifests"
```

Expected: commit succeeds.

---

### Task 4: Remove The Template Eval Harness

**Files:**
- Delete: `tests/evals/.gitignore`
- Delete: `tests/evals/package.json`
- Delete: `tests/evals/skill-eval-coverage-baseline.json`
- Delete: `tests/evals/prompts/skill-example-skill.txt`
- Delete: `tests/evals/packages/example-skill/skill-example-skill.yaml`
- Delete: `tests/evals/scripts/promptfoo.sh`
- Delete: `tests/evals/scripts/check-codex-compatibility.js`
- Delete: `tests/evals/scripts/check-skill-eval-coverage.js`
- Delete: `tests/evals/assertions/schema-helpers.js`
- Delete: `tests/evals/assertions/check-skill-contract.js`

- [ ] **Step 1: Remove eval harness files**

Run:

```bash
rm -rf tests/evals
```

Expected:

```bash
test ! -e tests/evals
```

- [ ] **Step 2: Confirm no eval commands remain in package files**

Run:

```bash
rg -n "promptfoo|eval:example-skill|tests/evals|skill-eval-coverage" . || true
```

Expected: matches may remain only in the design spec and implementation plan until documentation is updated in later tasks.

- [ ] **Step 3: Commit eval removal**

Run:

```bash
git add -A tests/evals
git commit -m "chore: remove template eval harness"
```

Expected: commit succeeds with all `tests/evals/` files removed.

---

### Task 5: Add Reviewed Upstream Sync Workflow

**Files:**
- Create: `.github/workflows/sync-upstream.yml`

- [ ] **Step 1: Create the workflow**

Create `.github/workflows/sync-upstream.yml` with:

```yaml
name: sync-upstream-frontend-design

on:
  workflow_dispatch:
  schedule:
    - cron: "17 10 * * 1"

permissions:
  contents: write
  pull-requests: write

jobs:
  sync:
    name: sync upstream frontend-design skill
    runs-on: ubuntu-latest
    steps:
      - name: Checkout wrapper repository
        uses: actions/checkout@v4

      - name: Checkout upstream plugin sparsely
        run: |
          set -eu
          git clone --depth 1 --filter=blob:none --sparse \
            https://github.com/anthropics/claude-plugins-official.git \
            /tmp/claude-plugins-official
          cd /tmp/claude-plugins-official
          git sparse-checkout set plugins/frontend-design

      - name: Copy upstream-owned files
        run: |
          set -eu
          rm -rf skills
          mkdir -p skills
          cp -R /tmp/claude-plugins-official/plugins/frontend-design/skills/. skills/
          cp /tmp/claude-plugins-official/plugins/frontend-design/LICENSE LICENSE
          cp /tmp/claude-plugins-official/plugins/frontend-design/README.md /tmp/upstream-frontend-design-README.md

      - name: Validate wrapper integrity
        run: |
          set -eu
          jq empty .claude-plugin/plugin.json
          jq empty .codex-plugin/plugin.json
          test -f skills/frontend-design/SKILL.md
          test ! -e skills/example-skill
          test ! -e tests/evals
          grep -R "anthropics/claude-plugins-official" .github/workflows/sync-upstream.yml
          grep -R "plugins/frontend-design" .github/workflows/sync-upstream.yml

      - name: Open sync pull request
        uses: peter-evans/create-pull-request@v6
        with:
          branch: automation/sync-upstream-frontend-design
          delete-branch: true
          commit-message: "chore: sync upstream frontend-design skill"
          title: "chore: sync upstream frontend-design skill"
          body: |
            Syncs upstream-owned files from:

            https://github.com/anthropics/claude-plugins-official/tree/main/plugins/frontend-design

            Copied files:
            - plugins/frontend-design/skills/ -> skills/
            - plugins/frontend-design/LICENSE -> LICENSE

            The wrapper-owned manifests, repository docs, and workflow files remain local.
          labels: |
            upstream-sync
```

- [ ] **Step 2: Validate workflow syntax by reading key markers**

Run:

```bash
rg -n "schedule|anthropics/claude-plugins-official|plugins/frontend-design|create-pull-request" .github/workflows/sync-upstream.yml
```

Expected output includes all four terms.

- [ ] **Step 3: Confirm the workflow does not push directly to main**

Run:

```bash
rg -n "push origin main|git push origin main" .github/workflows/sync-upstream.yml || true
```

Expected: no output.

- [ ] **Step 4: Commit sync workflow**

Run:

```bash
git add .github/workflows/sync-upstream.yml
git commit -m "ci: add upstream sync workflow"
```

Expected: commit succeeds.

---

### Task 6: Update Repository Guidance And Docs

**Files:**
- Modify: `README.md`
- Modify: `AGENTS.md`
- Modify: `repo-map.json`
- Confirm: `CLAUDE.md`

- [ ] **Step 1: Replace `README.md`**

Replace `README.md` with:

````markdown
# frontend-design

Accelerate Data wrapper repository for Anthropic's official `frontend-design` skill.

This repository is a single plugin source repo, not a marketplace repo. It packages the upstream skill as ordinary plugin files so Claude and Codex marketplaces can install the same whole-repo source.

## Upstream Source

The runtime skill content is sourced from Anthropic's official plugin repository:

```text
https://github.com/anthropics/claude-plugins-official/tree/main/plugins/frontend-design
```

Synced upstream-owned files:

- `skills/frontend-design/`
- `LICENSE`

Wrapper-owned files:

- `.claude-plugin/plugin.json`
- `.codex-plugin/plugin.json`
- `AGENTS.md`
- `CLAUDE.md`
- `README.md`
- `repo-map.json`
- `.github/workflows/sync-upstream.yml`
- local design and plan docs

## Marketplace Source

Marketplace entries should point at this wrapper repository as a whole-repo URL source:

```json
{
  "source": "url",
  "url": "https://github.com/accelerate-data/frontend-design.git"
}
```

Do not point Codex directly at Anthropic's upstream plugin path; the upstream plugin does not provide `.codex-plugin/plugin.json`.

## Local Development

For direct local use without a marketplace, symlink the skill directory:

```bash
mkdir -p ~/.claude/skills ~/.codex/skills

ln -s /absolute/path/to/frontend-design/skills/frontend-design \
  ~/.claude/skills/frontend-design

ln -s /absolute/path/to/frontend-design/skills/frontend-design \
  ~/.codex/skills/frontend-design
```

Enable the repo-managed pre-commit hook:

```bash
git config core.hooksPath .githooks
chmod +x .githooks/pre-commit
```

## Upstream Sync

The `sync-upstream-frontend-design` workflow runs weekly and can be dispatched manually. It sparse-checkouts `plugins/frontend-design` from `anthropics/claude-plugins-official`, copies upstream-owned runtime files into this wrapper repo, validates wrapper integrity, and opens a pull request for review.

The workflow must not push directly to `main`.

## Validation

Run these checks before opening or updating a pull request:

```bash
jq empty .claude-plugin/plugin.json
jq empty .codex-plugin/plugin.json
test -f skills/frontend-design/SKILL.md
test ! -e skills/example-skill
test ! -e tests/evals
rg -n "anthropics/claude-plugins-official|plugins/frontend-design|schedule" .github/workflows/sync-upstream.yml
```
````

- [ ] **Step 2: Replace `AGENTS.md` Skills and eval template guidance**

Update `AGENTS.md` so it keeps the existing behavior guidelines but changes the project section to:

````markdown
# frontend-design

Wrapper repository for Anthropic's official `frontend-design` skill, published through Accelerate Data Claude and Codex marketplaces.

**Maintenance rule:** This file contains durable repository guidance, not volatile inventory. If a fact is easy to rediscover from the tree or will go stale when files move, keep it in `repo-map.json` instead.

## Instruction Hierarchy

Use this precedence when maintaining agent guidance:

1. `AGENTS.md` - canonical, cross-agent source of truth
2. Upstream-owned skill content under `skills/frontend-design/`
3. `CLAUDE.md` - Claude-specific adapter and routing

For Codex, `AGENTS.md` is also the repo-local instruction surface. Do not add a separate Codex adapter file unless Codex introduces a real supported convention for one.

Adapter files must stay lightweight and should not duplicate canonical policy unless they add agent-specific behavior.

## Repository Purpose

This repository is a single plugin-source wrapper repo, not a marketplace repo.

- Root manifests: `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json`
- Runtime skill content: `skills/frontend-design/`
- Upstream source: `anthropics/claude-plugins-official`, path `plugins/frontend-design/`
- Sync workflow: `.github/workflows/sync-upstream.yml`

## Agent Startup Context

Read `repo-map.json` before any non-trivial task. It is the primary index for structure, commands, and the current wrapper layout.

## Upstream-Owned Content

The following files are synced from `anthropics/claude-plugins-official/plugins/frontend-design`:

- `skills/frontend-design/`
- `LICENSE`

AI agents may edit wrapper-owned files such as plugin manifests, sync workflow files, repository docs, validation commands, and local design or plan docs as normal.

Before editing synced upstream content under `skills/frontend-design/`, warn the user that the file is upstream-owned and explain that the change may diverge from future upstream syncs. Proceed only when the user explicitly confirms the local divergence or when the task is specifically to resolve an upstream sync conflict.

## Maintenance Rules

| Artifact | Update when |
|---|---|
| `AGENTS.md` | A fact is durable, cross-cutting, and not obvious from the code tree |
| `repo-map.json` | Any structural entry becomes stale: manifests, skills, sync workflow layout, commands, or key docs |
| `CLAUDE.md` | Claude-specific routing or adapter behavior changes |
| `README.md` | User-facing wrapper installation, attribution, or sync behavior changes |

Update stale guidance in the same change that introduces the structural change.

## Testing

This repository does not maintain promptfoo eval coverage for Anthropic-owned skill behavior.

Validation should verify wrapper integrity:

1. `.claude-plugin/plugin.json` is valid JSON.
2. `.codex-plugin/plugin.json` is valid JSON.
3. `skills/frontend-design/SKILL.md` exists.
4. `skills/example-skill/` does not exist.
5. `tests/evals/` does not exist.
6. `.github/workflows/sync-upstream.yml` references `anthropics/claude-plugins-official` and `plugins/frontend-design`.
7. `.github/workflows/sync-upstream.yml` has a weekly `schedule` trigger.

## Local Development Pattern

For direct local use without a marketplace, symlink `skills/frontend-design` into:

- `~/.claude/skills/frontend-design`
- `~/.codex/skills/frontend-design`

Keep the symlink name identical to the skill directory name.

## Git Hooks

This repo provides a repo-managed pre-commit hook in `.githooks/pre-commit`.

- Enable it with `git config core.hooksPath .githooks`
- Keep it focused on durable repo policy, not machine-local tooling

## Worktrees

Use sibling worktrees at `../worktrees/<branch-name>` relative to the repo root.

- Preserve the full branch name in the path, including prefixes such as `feature/`.
- After creating the worktree, run `./scripts/setup-worktree.sh ../worktrees/<branch-name>` from the main checkout.
- `scripts/setup-worktree.sh` derives paths relative to the repository and must stay portable across developers and machines.

## Skills

- `skills/frontend-design/SKILL.md` - upstream Anthropic frontend design skill

## Conventions

- Keep all skill directories under `skills/`.
- Do not add repo-specific styling, product assumptions, or external path dependencies to the upstream-owned skill.
- Marketplace changes belong in `plugin-marketplace`, not in this wrapper repo.
````

Keep the top-level `# Codex Behavior Guidelines` section already present in `AGENTS.md` before this project section.

- [ ] **Step 3: Replace `repo-map.json`**

Replace `repo-map.json` with:

```json
{
  "generated_at": "2026-04-25T00:00:00Z",
  "repo_root": "frontend-design",
  "primary_languages": ["Markdown", "JSON", "Shell", "YAML"],
  "package_managers": [],
  "test_frameworks": [],
  "applications": [
    {
      "name": "frontend-design plugin wrapper",
      "type": "shared plugin source",
      "description": "Accelerate Data wrapper around Anthropic's official frontend-design skill, with Claude and Codex manifests."
    }
  ],
  "modules": {
    "claude_manifest": {
      "path": ".claude-plugin/plugin.json",
      "description": "Claude plugin manifest for the frontend-design wrapper."
    },
    "codex_manifest": {
      "path": ".codex-plugin/plugin.json",
      "description": "Codex plugin manifest for the frontend-design wrapper."
    },
    "frontend_design_skill": {
      "path": "skills/frontend-design/",
      "description": "Upstream-owned Anthropic frontend-design skill copied from plugins/frontend-design/skills/frontend-design/."
    },
    "sync_workflow": {
      "path": ".github/workflows/sync-upstream.yml",
      "description": "Weekly and manual sparse upstream sync workflow that opens a pull request."
    },
    "scripts": {
      "path": "scripts/",
      "description": "Repository helper scripts. setup-worktree.sh prepares a newly created sibling worktree."
    },
    "docs": {
      "path": "docs/superpowers/",
      "description": "Local design specs and implementation plans for this wrapper repository."
    }
  },
  "key_directories": {
    "skills/frontend-design/": "Upstream-owned runtime skill content",
    "scripts/": "Repository helper scripts",
    ".claude-plugin/": "Claude plugin manifest",
    ".codex-plugin/": "Codex plugin manifest",
    ".github/workflows/": "Repository CI and upstream sync workflows",
    "docs/superpowers/": "Design specs and implementation plans"
  },
  "commands": {
    "setup_worktree": "./scripts/setup-worktree.sh ../worktrees/<branch-name>",
    "validate_manifests": "jq empty .claude-plugin/plugin.json && jq empty .codex-plugin/plugin.json",
    "validate_wrapper": "jq empty .claude-plugin/plugin.json && jq empty .codex-plugin/plugin.json && test -f skills/frontend-design/SKILL.md && test ! -e skills/example-skill && test ! -e tests/evals && rg -n \"anthropics/claude-plugins-official|plugins/frontend-design|schedule\" .github/workflows/sync-upstream.yml"
  },
  "conventions": {
    "skill_location": "The packaged runtime skill lives under skills/frontend-design/.",
    "upstream_owned_content": "skills/frontend-design/ and LICENSE are synced from anthropics/claude-plugins-official/plugins/frontend-design.",
    "wrapper_owned_content": "Plugin manifests, AGENTS.md, CLAUDE.md, README.md, repo-map.json, workflows, scripts, and docs are maintained in this wrapper repository.",
    "upstream_edit_warning": "Warn before editing skills/frontend-design/ because local edits may diverge from future upstream syncs.",
    "worktrees": "Use sibling worktrees at ../worktrees/<branch-name> and run scripts/setup-worktree.sh after creation.",
    "local_dev_symlinks": "For direct local use, symlink skills/frontend-design into ~/.claude/skills/frontend-design and ~/.codex/skills/frontend-design.",
    "marketplace_scope": "Marketplace entries belong in plugin-marketplace and should point to this wrapper repository as a URL source."
  },
  "notes_for_agents": {
    "startup": "Read this file before exploring the repository. It is the primary startup context for structure, ownership, and validation commands.",
    "marketplace_model": "This repository is a plugin source, not a marketplace. Marketplaces in other repositories should point to this repo as the plugin source.",
    "eval_scope": "This repository intentionally does not own promptfoo eval coverage for upstream Anthropic skill behavior.",
    "update_repo_map": "Refresh this file whenever manifests, skill layout, sync workflow layout, validation commands, or key docs change."
  }
}
```

- [ ] **Step 4: Confirm `CLAUDE.md` stays lightweight**

Run:

```bash
sed -n '1,80p' CLAUDE.md
```

Expected:

```text
@AGENTS.md

## Adapter Role

`AGENTS.md` is canonical for repository-wide guidance. This file is a Claude-specific adapter and should stay lightweight.
```

- [ ] **Step 5: Validate documentation no longer presents template instructions**

Run:

```bash
rg -n "Your Plugin Name|example-skill|Template repo|npm run eval|promptfoo|skill-eval-coverage" README.md AGENTS.md repo-map.json CLAUDE.md || true
```

Expected: no output.

- [ ] **Step 6: Commit docs and repo map**

Run:

```bash
git add README.md AGENTS.md repo-map.json CLAUDE.md
git commit -m "docs: document frontend-design wrapper"
```

Expected: commit succeeds.

---

### Task 7: Final Wrapper Validation

**Files:**
- Verify: `.claude-plugin/plugin.json`
- Verify: `.codex-plugin/plugin.json`
- Verify: `skills/frontend-design/SKILL.md`
- Verify: `.github/workflows/sync-upstream.yml`
- Verify: `README.md`
- Verify: `AGENTS.md`
- Verify: `repo-map.json`

- [ ] **Step 1: Run required wrapper validation**

Run:

```bash
jq empty .claude-plugin/plugin.json
jq empty .codex-plugin/plugin.json
test -f skills/frontend-design/SKILL.md
test ! -e skills/example-skill
test ! -e tests/evals
rg -n "anthropics/claude-plugins-official|plugins/frontend-design|schedule" .github/workflows/sync-upstream.yml
```

Expected: both `jq` and all `test` commands pass; `rg` shows matches for upstream repository, upstream path, and schedule trigger.

- [ ] **Step 2: Confirm marketplace source text is present**

Run:

```bash
rg -n '"source": "url"|https://github.com/accelerate-data/frontend-design.git|plugin-marketplace' README.md AGENTS.md docs/superpowers/specs/2026-04-25-frontend-design-wrapper-design.md
```

Expected: README shows the whole-repo URL source, and AGENTS/spec guidance says marketplace changes belong in `plugin-marketplace`.

- [ ] **Step 3: Confirm no template files remain**

Run:

```bash
rg -n "your-plugin-name|Your Plugin Name|example-skill|Template repo" . || true
```

Expected: no output, except historical references inside committed plans/specs if the implementation intentionally preserves context there. Do not edit upstream-owned `skills/frontend-design/SKILL.md` to satisfy this check.

- [ ] **Step 4: Review full diff**

Run:

```bash
git diff --stat HEAD~6..HEAD
git diff HEAD~6..HEAD -- .claude-plugin/plugin.json .codex-plugin/plugin.json README.md AGENTS.md repo-map.json .github/workflows/sync-upstream.yml
```

Expected: diff shows wrapper conversion only: manifests updated, template evals removed, upstream skill added, sync workflow created, and docs updated.

- [ ] **Step 5: Commit validation fixes if any were needed**

If Step 1, 2, or 3 required fixes, run:

```bash
git add -A
git commit -m "chore: validate frontend-design wrapper"
```

Expected: commit succeeds only if validation fixes were made. If no fixes were needed, skip this commit.

---

## Self-Review Checklist

- [ ] The plan replaces the template `skills/example-skill/` with `skills/frontend-design/`.
- [ ] The plan removes `tests/evals/` and does not add local promptfoo coverage.
- [ ] The plan keeps upstream-owned runtime content as ordinary files in the wrapper repo.
- [ ] The plan adds both Claude and Codex manifests for `frontend-design`.
- [ ] The plan adds a sparse upstream sync workflow that opens a PR instead of pushing to `main`.
- [ ] The plan updates `AGENTS.md` with upstream-owned content warning rules.
- [ ] The plan updates `repo-map.json` after structural changes.
- [ ] The plan validates manifests, skill presence, template skill removal, eval harness removal, sync workflow markers, and marketplace source text.
