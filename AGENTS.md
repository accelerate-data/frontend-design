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

This repository does not maintain local eval coverage for Anthropic-owned skill behavior.

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
