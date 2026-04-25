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
