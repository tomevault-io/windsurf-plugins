---
trigger: always_on
description: Open-source, file-based team-knowledge workspace desktop app powered by Claude Code. macOS, Apache 2.0.
---


## Quick reference

| What | Where |
|------|-------|
| GitHub | `openit-app/openit-app` |
| Landing page | `https://openit-app.github.io/openit-app` |
| Tech stack | React + TypeScript frontend, Rust (Tauri) backend |
| Tests | `npm test` (vitest), `cd src-tauri && cargo test` |
| Lint | `cargo fmt -- --check`, `cargo clippy` |
| Dev mode | `npm run tauri dev` |
| Production build | `npm run tauri build` |

## Releasing

Releases are fully automated. To cut a release:

```bash
# 1. Bump version in ALL THREE files (must match):
#    - src-tauri/tauri.conf.json
#    - src-tauri/Cargo.toml
#    - package.json
# 2. Update Cargo.lock:
cd src-tauri && cargo generate-lockfile && cd ..
# 3. Commit, tag, push:
git add -A && git commit -m "chore: bump version to X.Y.Z"
git push origin HEAD:main
git tag vX.Y.Z && git push origin vX.Y.Z
```

The release workflow (`.github/workflows/release.yml`) then:
1. Builds both DMGs (Apple Silicon + Intel) sequentially on one runner
2. Signs, notarizes, and uploads DMGs + updater archives
3. Signs the updater archives with `tauri signer` and builds `latest.json`
4. Uploads `latest.json` to the release (auto-updater endpoint)
5. The landing page auto-rebuilds on release publish (picks up new download links)

**Do NOT set `includeUpdaterJson: true` in tauri-action** — it's broken. The workflow builds `latest.json` itself in a separate step.

### Updater signing key

The updater uses a minisign keypair. The private key is in GitHub Secrets (`TAURI_SIGNING_PRIVATE_KEY`), the public key is in `src-tauri/tauri.conf.json` under `plugins.updater.pubkey`. These must match. If you regenerate the keypair:
1. `npm run tauri -- signer generate -w /tmp/key --ci -f`
2. Update the pubkey in `tauri.conf.json`
3. Update `TAURI_SIGNING_PRIVATE_KEY` in GitHub Secrets
4. Set `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` to empty string in GitHub Secrets

### In-app auto-updater

The app checks for updates on launch and every 5 minutes. When a new version is available, an "Update vX.Y.Z" button appears in the title bar. Users click it → download, install, relaunch. The updater fetches `latest.json` from the latest GitHub release.

## Architecture

```
src/                    # React frontend (TypeScript)
  shell/                # App shell — file explorer, workbench
    viewers/            # Entity-specific viewers (agent, datastore, conversation, etc.)
    explorer/           # File tree (TreeNode, ContextMenu, useTreeState)
    routing/            # Path → ViewerSource resolvers (per-entity-group)
  ui/                   # Design system components
  lib/                  # Core logic — API bindings, sync, catalogs, updater
src-tauri/src/          # Tauri backend (Rust)
  kb/                   # Knowledge base (local.rs + cloud.rs + types.rs)
  ...                   # PTY, file watching, git ops, tools
scripts/openit-plugin/  # Claude plugin — skills, scripts, schemas, seed data
landing/                # Website (Astro + Tailwind) → GitHub Pages
```

## Data model — primitives and stores

Everything in OpenIT is a file or folder on disk. The workstation organizes them into **primitives** (top-level container types) and **stores** (instances the user creates inside them).

### Primitives (always available)

| Primitive | Disk path | What lives inside |
|-----------|-----------|-------------------|
| Databases | `databases/` | JSON-row collections (people, access, assets, tickets, + user-created) |
| Filestores | `filestores/` | File collections (attachments, library, skills, scripts, + user-created) |
| Knowledge | `knowledge-bases/` | Markdown articles |
| Reports | `reports/` | Generated markdown reports |
| Agents | `agents/` | Agent definitions (.md files) |

### System entities (not user-customizable containers)

| Entity | Disk path | Notes |
|--------|-----------|-------|
| Tools | (synthetic) | Detected via `which`, no on-disk folder |
| Traces | `.openit/agent-traces/` | Auto-generated agent activity logs |
| Inbox | `databases/tickets/` + `databases/conversations/` | Handled by the hero card, not a standalone tile |

### Workstation config (`.openit/workstation.json`)

The workstation layout is fully customizable. Users (or Claude Code) can create, delete, promote, demote, and customize any store. The config file controls:

```json
{
  "main": [
    { "rel": "knowledge-bases" },
    { "rel": "filestores/skills" }
  ],
  "more": [
    { "rel": "databases/people", "icon": "person", "tone": "sage", "label": "People", "description": "Contacts directory" },
    { "rel": "databases/roles", "icon": "shield", "tone": "link", "label": "Roles" }
  ]
}
```

- **`rel`**: repo-relative path (the tile's identity)
- **`label`**: custom display name (overrides the default)
- **`icon`**: key from `ICON_GALLERY` in `entityIcons.tsx` (34 icons available)
- **`tone`**: color theme — `accent`, `sage`, `ochre`, `link`, `clay`, `neutral`
- **`description`**: short text shown on list-view cards

Tiles in `main` appear in the primary workstation area. Tiles in `more` appear in the collapsible "More" pool. Newly created stores auto-register in `more`. Deleted stores are auto-cleaned from the config.

### What can be deleted


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openit-app/openit-app](https://github.com/openit-app/openit-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
