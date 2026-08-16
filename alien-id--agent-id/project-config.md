---
trigger: always_on
description: Universal, verifiable identity for AI agents, optionally linked to a human owner
---

# Alien Agent ID

Universal, verifiable identity for AI agents, optionally linked to a human owner
via Alien Network SSO. Ships as a set of Claude Code marketplace plugins plus two
shared npm libraries.

## Project structure

Monorepo using **Bun workspaces**. Plain ESM `.mjs` — **no TypeScript build** for
the plugins (the only `.ts` is the release tooling under `scripts/`).

```
agent-id/
├── plugins/
│   ├── agent-id-core/     # @alien-id/agent-id-core  (npm, public) — crypto, v3 bundle, verifier, OIDC, state
│   ├── agent-id-vault/    # @alien-id/agent-id-vault (npm, public) — encrypted credential vault (→ core)
│   ├── agent-id-auth/     # marketplace-only (private) — DPoP calls (→ core)
│   ├── agent-id-git/      # marketplace-only (private) — SSH-signed commits (→ core)
│   ├── agent-id-proxy/    # marketplace-only (private) — credential-injecting proxy (→ core, vault)
│   └── agent-id-browser/  # marketplace-only (private) — vault-sealed browser (→ core, vault)
├── .claude-plugin/marketplace.json   # marketplace catalog (versions synced from package.json)
├── scripts/               # changesets release tooling (.ts, run with bun)
├── tests/                 # node --test suite (test-*.mjs)
└── .changeset/            # changesets config + pending release intents
```

### Two distribution layers

- **npm**: only `@alien-id/agent-id-core` and `@alien-id/agent-id-vault` are
  published. They are the shared libraries the other plugins import.
- **Claude Code marketplace**: all six plugins. `auth`/`git`/`proxy`/`browser`
  are `private` (never on npm) and consume the two libs at runtime.

### Cross-plugin code sharing

Plugins import shared code as **bare specifiers** (`@alien-id/agent-id-core/lib/*.mjs`),
never relative sibling paths. Resolution:

- **Dev**: bun workspaces symlink `@alien-id/*` to the local `plugins/*`.
- **Installed (marketplace)**: a `SessionStart` hook (`plugins/<name>/hooks/install-deps.sh`)
  `npm install`s the libs into `$CLAUDE_PLUGIN_DATA` (persistent) and symlinks
  `node_modules` into the ephemeral `$CLAUDE_PLUGIN_ROOT` so the bare imports
  resolve. This is why core/vault must be published before the other plugins can
  install.

  This is the official Claude Code **"Persistent data directory"** pattern
  (Plugins reference): a `SessionStart` hook that `npm install`s into
  `$CLAUDE_PLUGIN_DATA`. The docs pair it with `NODE_PATH`, but `NODE_PATH` is
  CJS-only and our plugins are ESM (plus one CJS `createRequire` for
  `qrcode.cjs`), so we substitute a regenerated `node_modules` **symlink** — one
  mechanism that resolves both. `$CLAUDE_PLUGIN_ROOT` is treated as ephemeral as
  the docs require: the symlink is a pointer (not persistent state), recreated
  each session, with all real packages in `$CLAUDE_PLUGIN_DATA`, so it survives
  plugin updates and never clobbers a real dev `node_modules`.

Internal deps pin `^semver` (e.g. `^7.0.0`), **not** `workspace:*` — the
marketplace installer uses `npm`, which can't resolve the `workspace:` protocol.

## Development

Use **Bun**.

```bash
bun install            # install + link workspaces
bun run test           # node --test tests/test-*.mjs (the plugin test suite)
bun test scripts/tests # the release-tooling unit tests
```

Per-plugin CLIs run directly in dev (workspace symlinks are in place):

```bash
node plugins/agent-id-core/bin/cli.mjs --help
node plugins/agent-id-vault/bin/cli.mjs --help
```

## Releases

Uses [changesets](https://github.com/changesets/changesets) in canonical
bot-driven mode. See `RELEASING.md` for the full mental model and the
repo-scoped `/release` skill to drive it. The shape:

1. A feature PR touching `plugins/agent-id-core/**` or `plugins/agent-id-vault/**`
   includes a `.changeset/*.md` (add via `bun changeset`). Changes to only the
   private plugins need no changeset.
2. On push to `main`, `changesets/action` opens/updates a `chore: release
   packages` Version PR. `ci:version` runs `changeset version && bun install &&
   bun run sync-plugin-versions`, so the PR also carries the propagated
   `plugin.json` / `marketplace.json` versions.
3. Merging the Version PR triggers the publish path, gated by the `npm-publish`
   environment reviewer.
4. `scripts/publish-topological.ts` walks the published packages dependency-first
   (`core → vault`), packs each with `bun pm pack`, and uploads via `npm publish
   <tgz> --provenance` (OIDC trusted publishing, sigstore attestation, no
   `NPM_TOKEN`). Idempotent via `npm view`; `changeset tag` runs after.
5. `bun pm pack` (not `bun publish`) because `bun publish` lacks `--provenance`.
   There is **no build step** — the plugins are raw `.mjs`.
6. **Version sync**: `scripts/sync-plugin-versions.ts` is the single propagation
   point from each `plugins/*/package.json` version into that plugin's
   `.claude-plugin/plugin.json` and the `marketplace.json` entry. `package.json`
   is the source of truth; CI fails on any drift.

**Bootstrap (one-time):** `@alien-id/agent-id-core@7.0.0` and
`@alien-id/agent-id-vault@7.0.0` must exist on npm before the dependent
marketplace plugins can install. Operator setup: configure npm trusted publishers
for both packages to authorize `release.yml`, and create the `npm-publish`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alien-id/agent-id](https://github.com/alien-id/agent-id) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
