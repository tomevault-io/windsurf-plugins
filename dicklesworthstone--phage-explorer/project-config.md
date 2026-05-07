---
trigger: always_on
description: You may NOT delete any file or directory unless I explicitly give the exact command **in this session**.
---

# Phage Explorer — Agent Guidelines

## RULE 1 – ABSOLUTE (DO NOT EVER VIOLATE THIS)

You may NOT delete any file or directory unless I explicitly give the exact command **in this session**.

- This includes files you just created (tests, tmp files, scripts, etc.).
- You do not get to decide that something is "safe" to remove.
- If you think something should be removed, stop and ask. You must receive clear written approval **before** any deletion command is even proposed.

Treat "never delete files without permission" as a hard invariant.

---

## IRREVERSIBLE GIT & FILESYSTEM ACTIONS

Absolutely forbidden unless I give the **exact command and explicit approval** in the same message:

- `git reset --hard`
- `git clean -fd`
- `rm -rf`
- Any command that can delete or overwrite code/data

Rules:

1. If you are not 100% sure what a command will delete, do not propose or run it. Ask first.
2. Prefer safe tools: `git status`, `git diff`, `git stash`, copying to backups, etc.
3. After approval, restate the command verbatim, list what it will affect, and wait for confirmation.
4. When a destructive command is run, record in your response:
   - The exact user text authorizing it
   - The command run
   - When you ran it

If that audit trail is missing, then you must act as if the operation never happened.

---

## Project Stack & Toolchain

### Node / JS / TS

- Use **bun** for everything JS/TS.
  - ❌ Never use `npm`, `yarn`, or `pnpm`.
  - Lockfiles: only `bun.lock`. Do not introduce any other lockfile.
- Target **latest Node.js** (local + Vercel). No need to support old Node versions.

### Monorepo Structure

This is a monorepo with packages:

| Package | Purpose |
|---------|---------|
| `packages/core` | Core analysis algorithms, types, utilities |
| `packages/web` | React web app (Vite + React) |
| `packages/tui` | Terminal UI (Ink + React) |
| `packages/comparison` | Phage comparison algorithms |
| `packages/db-runtime` | Database runtime (Bun SQLite) |
| `packages/db-schema` | Database schema definitions |
| `packages/data-pipeline` | NCBI data fetching and processing |
| `packages/renderer-3d` | 3D visualization (Three.js) |
| `packages/state` | Zustand state management |
| `packages/wasm-compute` | Rust/WASM performance-critical code |

### Rust / WASM

The `packages/wasm-compute` package contains Rust code compiled to WASM:

- Build with `wasm-pack build --target web`
- WASM is inlined into JS during build for Vercel deployment
- Use Rust for compute-intensive operations (dot plots, k-mer analysis, etc.)
- Keep Rust code minimal and focused on hot paths

### Deployment

- Deployed to Vercel
- SQLite database shipped as static asset (`phage.db`)
- sql.js used for browser-side SQLite queries

---

## Code Editing Discipline

- Do **not** run scripts that bulk-modify code (codemods, invented one-off scripts, giant `sed`/regex refactors).
- Large mechanical changes: break into smaller, explicit edits and review diffs.
- Subtle/complex changes: edit by hand, file-by-file, with careful reasoning.

---

## Backwards Compatibility & File Sprawl

We optimize for a clean architecture now, not backwards compatibility.

- No "compat shims" or "v2" file clones.
- When changing behavior, migrate callers and remove old code **inside the same file**.
- New files are only for genuinely new domains that don't fit existing modules.
- The bar for adding files is very high.

---

## Logging & Console Output

- Prefer a shared logger (e.g., `lib/logger.ts` on `pino`/`consola`) over raw `console.log`.
- No random console logs in UI components; if needed, make them dev-only and clean them up.
- Log structured context: IDs, user, request, model, etc.
- If a logger helper exists, you must use it; do not invent a different pattern.

---

## React Patterns (Web & TUI)

### Overlays

The web app uses a consistent overlay pattern in `packages/web/src/components/overlays/`:

- Each overlay is a lazy-loaded React component
- Register overlays in `OverlayManager.tsx` and add to `OverlayId` type in `OverlayProvider.tsx`
- Use `useHotkey` hook for keyboard shortcuts
- Follow the loading state pattern:
  ```tsx
  if (!repository || !currentPhage) {
    setData([]);
    setLoading(false);  // Always clear loading on early return!
    return;
  }
  ```

### State Management

- Use Zustand stores from `@phage-explorer/state`
- Access via hooks: `usePhageStore(s => s.currentPhage)`
- Avoid prop drilling; prefer store selectors

---

## Third-Party Libraries

When unsure of an API, look up current docs (late-2025) rather than guessing.

---

## MCP Agent Mail — Multi-Agent Coordination

Agent Mail is already available as an MCP server; do not treat it as a CLI you must shell out to. MCP Agent Mail *should* be available to you as an MCP server; if it's not, flag to the user. They might need to start Agent Mail using the `am` alias or by running `cd "<directory_where_they_installed_agent_mail>/mcp_agent_mail" && bash scripts/run_server_with_token.sh` if the alias isn't available.

What Agent Mail gives:

- Identities, inbox/outbox, searchable threads.
- Advisory file reservations (leases) to avoid agents clobbering each other.
- Persistent artifacts in git (human-auditable).

Core patterns:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/phage_explorer](https://github.com/Dicklesworthstone/phage_explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
