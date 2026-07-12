---
trigger: always_on
description: Solo / personal project named **Casual Editor**. The path contains `melp/` as a folder name only — **not** a company or product. Do not call this project "melp" or imply organizational context.
---

# CLAUDE.md — Casual Editor

## What this repo is

Solo / personal project named **Casual Editor**. The path contains `melp/` as a folder name only — **not** a company or product. Do not call this project "melp" or imply organizational context.

A casual, real-time collaborative `.docx` editor, built on a local fork of `eigenpal/docx-editor` (MIT, React + ProseMirror with OOXML-preserving model). Real-time sync, presence, and snapshots are provided by the shared **Node** collab server **`CasualOffice/collab`** (Hocuspocus + Yjs on Fastify — a format-agnostic server also powering Casual Sheets), which the editor reaches through `HocuspocusProvider`. Document persistence is delegated to a pluggable host integration (WOPI / JWT-API). _The collab server (vendored at `./collab`) now also owns the REST surface — share-link/seed (`/api/rooms`), auth, files, WOPI — and serves the SPA. The legacy in-repo Go y-websocket gateway under `backend/` was **removed** 2026-06-28; **all sync / presence / persistence / REST work goes to the Node collab server.**_

## Architecture

```
Browser
   ├─ <DocxEditor> (our fork of eigenpal/docx-editor, MIT, in docx-editor/)
   │      schema: ProseMirror, layout: their layout-painter (preserves OOXML)
   ├─ y-prosemirror ySyncPlugin + yCursorPlugin (presence)
   └─ Y.Doc  ⇄  HocuspocusProvider (y-websocket protocol over WS)
                       │
                       ▼
   Collab server — Node / TypeScript (CasualOffice/collab, SEPARATE repo) — STATELESS
   ├─ Hocuspocus WS server on Fastify — one in-memory Y.Doc per live room,
   │  dropped when the last client disconnects
   ├─ Auth + WOPI + pluggable storage hooks
   └─ Snapshot / versioning on room drain (Y.Doc → .docx)
                       │
                       ▼
   Storage host (external, pluggable)
   - WOPI host (GetFile / PutFile) or JWT-secured REST API
```

**Stateless invariant:** the collab server has no DB and no on-disk update log. Document persistence is owned by the host. Its only state is the in-memory Y.Doc for currently-active sessions — gone when all clients disconnect, gone again on process restart (the host re-seeds).

## Working rules for Claude in this repo

1. **Never write technical claims about external systems from memory.** Read the actual source first; cite file paths.
2. **The editor is a fork we modify.** When filling fidelity gaps in the editor (text-box rendering is the known weak spot): write a Playwright test reproducing the gap, fix in the right place per `docx-editor/CLAUDE.md`'s "Key File Map", open a PR upstream. Fork-and-diverge only if upstream rejects or stalls.
3. **Yjs + `y-prosemirror` is the chosen CRDT.** Do not propose Automerge/Loro/custom alternatives without explicit user direction.
4. **MIT only on the editor side.** The AGPL `@eigenpal/docx-editor-agents` package and everything that depended on it has been removed from our fork. Do not reintroduce. (The Node `CasualOffice/collab` server is permissive; fine.)
5. **Editor toolchain is Bun.** `bun install`, `bun run dev` (localhost:5173), `bun run build`, `bun run typecheck`. Tests via `npx playwright test`. Bun is installed locally (1.3.x) so verify-before-ship works.
6. **The collab server is Node, not Go.** Real-time sync/presence/snapshots AND the REST surface (`/api/rooms`, `/auth`, `/files`, `/wopi`) are owned by the shared **Node/TypeScript** server `CasualOffice/collab` (Hocuspocus + Yjs on Fastify), vendored as the `./collab` submodule. Never describe the backend as Go — the old in-repo Go gateway (`backend/`) was removed 2026-06-28.
7. **No live document model on the server.** Y.Doc updates in, updates out. Snapshots produced on room drain.
8. **Default new editor-side code to the fork** (`docx-editor/`); new sync / presence / persistence / REST work goes to the **`CasualOffice/collab`** Node server (the `./collab` submodule).
9. **Don't install software via `curl | bash` from a remote URL without explicit user consent.** Use Homebrew, npm, or other reviewable package managers; ask the user which install method they prefer before running.
10. **Docs are first-class.** When a doc-tracked fact changes (status block, fidelity score, working set, milestone state), update the relevant doc in the same commit or right after. Stale docs poison every future session that opens them.

## Where things live

- `docx-editor/` — working fork of `eigenpal/docx-editor`. **Inlined into this repo** (no separate `.git/`; tracked as part of the outer repo per the `.gitignore`). AGPL `agent-use` package and dependents purged. Push to `git@github.com:schnsrw/docx.git`.
- **Collab server** — the **Node/TypeScript** `CasualOffice/collab` repo (Hocuspocus + Yjs on Fastify): real-time sync, presence, auth, WOPI, snapshots/versioning. The editor connects via `HocuspocusProvider` (`docx-editor/packages/react/src/collab/useCollab.ts`). This is THE backend for collaboration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CasualOffice/docs](https://github.com/CasualOffice/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
