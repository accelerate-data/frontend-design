# Frontend Design Wrapper Design

## Context

`accelerate-data/frontend-design` is a plugin-source repository intended to
publish Anthropic's official `frontend-design` skill through the Accelerate Data
Claude and Codex marketplaces.

The upstream source is:

```text
https://github.com/anthropics/claude-plugins-official
plugins/frontend-design/
```

The upstream plugin has a Claude manifest at
`plugins/frontend-design/.claude-plugin/plugin.json`, but it does not have a
Codex manifest. The runtime skill content is portable: its `SKILL.md` is
narrative frontend design guidance with no Claude-only tools, slash commands,
scripts, `.claude/` paths, or `allowed-tools` metadata. The only runtime
Claude-specific wording is a branding sentence about Claude's creative ability.

## Goals

- Package the official Anthropic `frontend-design` skill as an Accelerate
  Data-maintained plugin source.
- Support both Claude and Codex marketplaces from the same repository.
- Preserve upstream skill content as ordinary files under root `skills/`.
- Use sparse upstream sync to avoid vendoring the entire
  `claude-plugins-official` repository.
- Make upstream-owned content boundaries explicit to AI agents.
- Require agents to warn before editing synced upstream skill content.
- Remove template eval harness files. This repository should package and sync
  Anthropic's official skill, not maintain independent promptfoo coverage for it.

## Non-Goals

- Fork or rewrite the frontend design skill behavior.
- Vendor all official Anthropic plugins.
- Use sparse checkout as the marketplace install mechanism.
- Automatically merge upstream changes without review.
- Maintain local eval suites for upstream-owned skill behavior.

## Repository Shape

The wrapper repository should expose a normal plugin root:

```text
frontend-design/
  .claude-plugin/plugin.json
  .codex-plugin/plugin.json
  AGENTS.md
  README.md
  LICENSE
  skills/
    frontend-design/
      SKILL.md
  .github/workflows/sync-upstream.yml
```

The existing template `skills/example-skill/` should be removed when the real
`skills/frontend-design/` directory is added.

The existing template eval harness under `tests/evals/` should also be removed.
Validation should focus on wrapper integrity and upstream sync correctness,
because the skill behavior itself is owned by Anthropic.

## Upstream Sync Strategy

Sparse checkout is appropriate for the sync workflow, not for marketplace
installation.

The sync workflow should:

1. Clone `anthropics/claude-plugins-official` into a temporary directory.
2. Use sparse checkout for `plugins/frontend-design/**`.
3. Copy upstream runtime files into this wrapper repo:
   - `plugins/frontend-design/skills/` -> `skills/`
   - `plugins/frontend-design/LICENSE` -> `LICENSE`
   - `plugins/frontend-design/README.md` -> upstream section or reference copy
4. Preserve wrapper-owned files:
   - `.claude-plugin/plugin.json`
   - `.codex-plugin/plugin.json`
   - `AGENTS.md`
   - `.github/workflows/sync-upstream.yml`
   - local design and plan docs
5. Open a pull request for review.

The workflow should run weekly and should not push directly to `main`.

## Marketplace Identity

The plugin name remains `frontend-design`. Both marketplace entries should point
to the wrapper repository as a whole-repo source:

```json
{
  "source": "url",
  "url": "https://github.com/accelerate-data/frontend-design.git"
}
```

Claude no longer needs to point directly at
`anthropics/claude-plugins-official/plugins/frontend-design` after this wrapper
repo contains the skill and manifests. Codex should not point directly at the
official Anthropic plugin because the upstream plugin lacks
`.codex-plugin/plugin.json`.

## Plugin Manifests

The wrapper should provide both manifests:

- `.claude-plugin/plugin.json`
- `.codex-plugin/plugin.json`

Both should use:

- `name`: `frontend-design`
- `description`: frontend design skill for production UI/UX implementation
- `author`: Accelerate Data, with upstream attribution in README
- `repository`: `https://github.com/accelerate-data/frontend-design`

The README should clearly attribute the skill source to Anthropic's official
plugin repository and link to the upstream path.

## Agent Safety Note

`AGENTS.md` should state that `skills/frontend-design/`, copied upstream README
content, and copied upstream license text are synced from
`anthropics/claude-plugins-official/plugins/frontend-design`.

AI agents may edit wrapper-owned files such as plugin manifests, sync workflow
files, repository docs, and validation scripts as normal.

Before editing synced upstream content under `skills/frontend-design/`, agents
must warn the user that the file is upstream-owned and explain that the change
may diverge from future upstream syncs. They may proceed only when the user
explicitly confirms the local divergence or when the task is specifically to
resolve an upstream sync conflict.

## Validation

The implementation should verify:

- `.claude-plugin/plugin.json` is valid JSON.
- `.codex-plugin/plugin.json` is valid JSON.
- `skills/frontend-design/SKILL.md` exists.
- `skills/example-skill/` no longer exists.
- `tests/evals/` no longer exists.
- The sync workflow text references `anthropics/claude-plugins-official` and
  `plugins/frontend-design`.
- The sync workflow has a weekly `schedule` trigger.
- The plugin can be referenced from `plugin-marketplace` as a `url` source.

Marketplace changes belong in `plugin-marketplace`, not in this wrapper repo.
