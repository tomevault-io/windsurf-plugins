---
trigger: always_on
description: | Task | Read First |
---

# Vibes DIY Plugin - Development Guide

## Agent Quick Reference

### When to Read What

| Task | Read First |
|------|------------|
| Working on skills | The specific `skills/*/SKILL.md` file |
| Generating app code | SKILL.md has TinyBase patterns and common hooks |
| Working on scripts | `scripts/package.json` for deps |
| Debugging React errors | `.claude/rules/react-singleton.md` loads automatically; also `skills/vibes/SKILL.md` Common Mistakes |
| Deploying to Cloudflare | `skills/cloudflare/SKILL.md` |
| Testing plugin changes | `cd scripts && npm test` for all tests; `/vibes:test` for full E2E |
| Editing auth components | `.claude/rules/auth-components.md` loads automatically |
| Editing templates or build system | `.claude/rules/template-build.md` loads automatically |
| Working on sharing/invites | `.claude/rules/sharing-architecture.md` loads automatically |

### TinyBase API Reference

SKILL.md provides common patterns (useTable, useRow, useAddRowCallback, store.setRow/delRow) and critical gotchas.

TinyBase hooks are exposed as `window.*` globals in the template (useTable, useRow, useCell, useRowIds, useSortedRowIds, useAddRowCallback, etc.). The `useApp()` hook provides `{ isReady, isSyncing, user }` context.

### Environment Variables in SKILL.md

`CLAUDE_PLUGIN_ROOT` is set by plugin runtime but may be missing in dev mode (`claude --plugin .`). `CLAUDE_SKILL_DIR` is text-substituted before the agent sees the markdown — always reliable.

All SKILL.md bash blocks use the fallback pattern:
```bash
VIBES_ROOT="${CLAUDE_PLUGIN_ROOT:-$(dirname "$(dirname "${CLAUDE_SKILL_DIR}")")}"
```

`CLAUDE_SKILL_DIR` is `<plugin-root>/skills/<name>/`, so `dirname dirname` gives the plugin root.

## Critical Rules

### `?external=` for React Singleton

Any esm.sh package that depends on React MUST use `?external=react,react-dom`. Details in `.claude/rules/react-singleton.md` (loads automatically when editing templates).

### Import Map Lives in Base Template

The authoritative import map is in `source-templates/base/template.html`. After editing, run `bun scripts/merge-templates.js --force`.

### Skills Are Atomic

Each skill is ONE plan step — never decompose into sub-steps. Always invoke the skill before running its commands, even for reassembly/redeploy.

## Package Versions

The import map in `source-templates/base/template.html` is the authoritative source for current package versions (TinyBase from esm.sh, `oauth4webapi`, React 19.2.4). The OIDC bridge (`bundles/oidc-bridge.js`) is loaded as a local bundle for private app auth.

## Deploy Workflow

Apps deploy to Cloudflare Workers via the shared Deploy API Worker. No wrangler installation or user Cloudflare tokens required.

```bash
bun scripts/deploy-cloudflare.js --name <app> --file index.html
```

Auth happens automatically: the CLI opens a browser for Pocket ID login and caches credentials at `~/.vibes/auth.json`. The Deploy API accepts the assembled HTML plus an OIDC token and handles all Cloudflare API calls server-side, including:

- **Worker deployment**: Deploys the app as a Worker in a Workers for Platforms namespace.
- **WebSocket URL injection**: The Deploy API injects the `wsUrl` for TinyBase sync into the app HTML before deploying.
- **Sync**: Handled server-side by TinyBase Durable Objects. The DO auto-creates on first WebSocket connection — no provisioning step needed.

## Architecture: JSX + Babel

The plugin uses JSX with Babel runtime transpilation. See `source-templates/base/template.html` for the `<script type="text/babel">` pattern.

## Local Development

```bash
claude --plugin .                        # From the plugin directory
claude --plugin /path/to/VibesOS     # Or with absolute path
```

## Restarting the Preview Server

After editing server code, handlers, or templates (e.g. `scripts/server/`, `skills/vibes/templates/editor.html`), the running server must be restarted to pick up changes. The server auto-kills any existing process on the same port — just re-run the start command:

```bash
VIBES_ROOT="${CLAUDE_PLUGIN_ROOT:-$(pwd)}"
bun "$VIBES_ROOT/scripts/server.ts" --mode=editor
```

Run in background if you need to continue working:
```bash
bun "$VIBES_ROOT/scripts/server.ts" --mode=editor &
```

**Do NOT use `pkill -f server.ts`** — the server handles takeover automatically via `killProcessOnPort()`. Re-running the command is the only correct restart method.

The `--mode=editor` flag is required for the editor UI. Omit it for preview-only mode. Optional flags: `--port 3333` (default), `--prompt "..."`.

## Testing

```bash
cd scripts
npm install          # First time
npm test             # All tests
npm run test:unit    # Unit only (<1 second)
npm run test:integration  # Mocked external services
npm run test:e2e:server   # E2E local server for manual testing
```

### Integration Testing

| What Changed | How to Test |
|-------------|-------------|
| Full E2E (assembly + deploy + browser) | `/vibes:test` |

### E2E with /etc/hosts

For subdomain routing tests, add to `/etc/hosts`:
```
127.0.0.1  test-app.local  tenant1.test-app.local  admin.test-app.local
```
Then `npm run test:e2e:server` and open `http://test-app.local:3000`.

## Non-Obvious Files

| File | Why it matters |
|------|---------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [popmechanic/VibesOS](https://github.com/popmechanic/VibesOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
