---
trigger: always_on
description: Only contents the LLM cannot derive from the source — hard constraints / the Why behind counter-intuitive decisions / pitfalls already hit. Architecture descriptions point at source, they do not restate the implementation.
---

# Orkas architecture and layering

Only contents the LLM cannot derive from the source — hard constraints / the Why behind counter-intuitive decisions / pitfalls already hit. Architecture descriptions point at source, they do not restate the implementation.

---

## 1. Project shape

Single-process Electron desktop app: main = Node backend, renderer = vanilla HTML/CSS/JS, IPC for communication, local file storage. Startup: `bootstrap.cjs` → tsx loader → `src/main/index.ts`; no build step.

**Hard constraints**:
- main runs no HTTP / occupies no port / has no auth.
- The renderer talks through the `contextBridge`-exposed `window.orkas.{invoke, stream}` allow-list API; **do not introduce** TS / JSX / webpack / vite.
- Preload **must be `.js`** (the preload loader does not run the tsx hook); path is `src/main/preload.js`.
- All LLM calls go through the in-process `core-agent` (`import('#core-agent')` dynamic load); no subprocesses. **Why:** avoid IPC serialization; locks / cancellation / event streams share memory.
- Storage is JSON / JSONL primarily; sqlite is used in exactly one place — the KB vector store. **Why:** user data must stay readable, portable, and friendly to cloud sync (single file = single sync unit).
- **skill / agent / contexts are three first-class citizens**; multi-agent collaboration follows the §5 group-chat architecture, with **no more "main agent calls subagent over RPC"**.
- npm dependency allow-list is in `PC/package.json` (key entries: `electron / pi-ai / better-sqlite3 / sqlite-vec / fastembed / onnxruntime-node / pdfjs-dist / pdf-lib / mammoth / jimp`). **New dependencies require a discussion first.**
- Renderer-side third-party JS/CSS goes through static assets at `src/renderer/vendor/<name>/`; not via npm. **Why:** `require` is unavailable inside the contextBridge sandbox; routing through npm is actually a detour.
- **Cross-platform**: macOS + Windows are both primary (Linux is community-grade). New code prefers cross-platform implementations (Node stdlib); platform branches must be verified on real machines for each branch — getting one platform working is not enough.

---

## 2. Directory layout

```
PC/                          Electron project root, sole dev and packaging entry
├── bootstrap.cjs            Registers the tsx loader → require('./src/main')
├── data/                    Runtime data (gitignored, see §4)
├── userWorkSpace/           Default workspace for the main conversation (gitignored)
├── src/main/                Node backend (TS, transpiled at runtime by tsx)
│   ├── index.ts             Electron lifecycle + IPC registration
│   ├── preload.js           contextBridge → window.orkas (must be .js)
│   ├── paths.ts             **Single source of truth for paths**; never scatter hard-coded paths
│   ├── ipc/                 IPC handlers (see §3)
│   ├── features/            Business layer (users / chats / group_chat / skills / agents / contexts / kb_* / auth / permissions / ...)
│   ├── model/               Model-call layer (in-process core-agent)
│   ├── prompts/             *.md templates
│   └── util/                Pure functions (locks / path-sandbox / extract-* / file_to_chunks / ...)
├── src/renderer/            Frontend UI (vanilla, see §8)
├── src/core-agent/          AgentRunner / providers / PersistentSession / SkillLoader
└── src/builtin/skills/      Built-in skill source (synced by hash to data/builtin/skills/ on startup)
```

**Runtime data location**: dev = `PC/data/` + `PC/userWorkSpace/`; packaged = `<container>/{data,userWorkSpace}/`, where the container is chosen as macOS/Linux → `~/.orkas/`, Windows → the lowest-letter non-system fixed drive `<drive>:\.orkas\` (falling back to `C:\` if none). Full drive selection logic lives in `src/main/packaged-data-root.ts`.

---

## 3. Layering constraints

```
ipc/                IPC handlers: arg validation + call into features; no IO, no business logic
features/           Business layer: orchestrates storage + model + prompts; knows nothing about IPC
model/              Model-call layer; client.ts re-exports, implementation in model/core-agent/
model/core-agent/   Local adapters + tool overrides
storage.ts          File IO helpers (stdlib only)
prompts/            Template loader (stdlib only)
i18n.ts             UI language table lookup (stdlib + locales/*.json only; never imports features / model)
util/               Pure-function utilities (stdlib only or single third-party dep; **never reverse-import features/model**)
```

**Require rules**:
- `index.ts` / `ipc/` → `features/` / `storage` / `paths` / `prompts`
- `features/` → `storage` / `paths` / `prompts` / `model` / `util` / sibling features
- `model/core-agent/` → dynamic `import('#core-agent')`; locks via `util/locks`; **never read or write business data under data/** (only session jsonl). **Why:** the model layer is stateless; orchestration of business state lives only in features. The model layer touching business data = double-write = state desync.

**Key model/core-agent constraints** (each *-tool.ts has a header comment with the implementation details):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Orkas-AI/Orkas](https://github.com/Orkas-AI/Orkas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
