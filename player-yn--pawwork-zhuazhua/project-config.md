---
trigger: always_on
description: **Product:** Chrome MV3 **Selection-first Web Agent**.
---

# 爪爪 · Paw Work — Agent spine (Session Workspace Runtime)

**Product:** Chrome MV3 **Selection-first Web Agent**.  
Paw Mode select on Live Web + describe outcome → verified deliverable.  
Brand: quiet, precise, results-first.

**Install:** extension only. Storage keys remain `pagewand_*`.  
**Contract:** [docs/SESSION_WORKSPACE_RUNTIME.md](docs/SESSION_WORKSPACE_RUNTIME.md) · [docs/WORKSPACE_RUNTIME_AUTHORITY.md](docs/WORKSPACE_RUNTIME_AUTHORITY.md) · [docs/RUNTIME_STACK.md](docs/RUNTIME_STACK.md) · [docs/CWS_CODE_RUNTIME.md](docs/CWS_CODE_RUNTIME.md)  
**Entry:** `src/agent/vnext/runSession.product.js` · offscreen `SessionWorkspaceService` · `npm run test:session-workspace`  
**Narrative / design:** [`product/`](product/) · [`design-system/`](design-system/) · [`src/sidepanel/css/tokens.css`](src/sidepanel/css/tokens.css)

This file is the auto-loaded spine. Keep it short.

---

## Baseline

| Fact | Detail |
|------|--------|
| **Product** | Session Workspace Runtime |
| **Stable** | **`main`** · `C:\Users\yyy\Desktop\PawWork` |
| **Dev** | **`runtime-vnext`** · `C:\Users\yyy\Desktop\PawWork-vnext` |
| **Remote** | https://github.com/Player-YN/PawWork-vnext (private) · default **`main`** |
| **Public** | https://github.com/Player-YN/PawWork_ZhuaZhua · `Desktop\PawWork_ZhuaZhua` · **tree sync**, never push private history |
| **Release** | Merge **`runtime-vnext` → `main`** in `Desktop\PawWork` (never checkout `main` here); `git push origin main`; `npm run sync:public -- --commit --push`. Passerby install until CWS: clone public branch **`unpacked`** (not source); same bytes as GitHub Release `paw-work-unpacked.zip` (workflow `release-unpacked` on the **public** repo force-updates only `unpacked`; secret `PAW_TLDRAW_LICENSE_KEY`, never commit the key). CWS zip = same `pack:extension`. Keep developing on `runtime-vnext`. |

---

## North star

```text
SELECT + DESCRIBE OUTCOME → DELIVER
```

```text
Sidepanel
  → workspaceRpc (background)
  → Offscreen SessionWorkspaceService
  → every user message: sendMessage
  → AI SDK 7 ToolLoopAgent (toolChoice=auto)
  → tools always on: inspect / acquire / run / clarify / sheet / deck / doc / web (inventory aims targets, does not hide tools)
  → /artifacts (durable) + /scratch (per-execution, settled away)
  → SelectionGroups = ambient bound index; inspect on demand
  → packaged QuickJS + esbuild-wasm + AI SDK loader
  → 制表: Univer live grid (Facade/Command; full IWorkbookData persist)
  → 制图: tldraw Design/Slides (`pawCanvas`); HTML is a website or document, not a layout engine
  → 文字: Univer Docs and/or `data-paw-kind=document` HTML preview
  → capabilities may register from host plugins or external MCP (catalog + invoke; not 200 model tools)
```

Session is the workspace. Model decides answer vs deliverable. Tools are capabilities, not obligations. Capture ≠ full truth. Host enforces isolation, auth, and GC. Model cannot mutate SelectionGroups. Live canvases (sheet / design / site / docs) are first-class work surfaces, not optional renderers. Sidepanel may run **one agent per Session in parallel**; switching Task does not abort the other. Capture Groups + items are ambient (same picker across tasks). Bind, artifacts, clipboard, and live office tabs stay session-scoped (empty sessionId never paints the foreground).

---

## Repo map

```text
PawWork-vnext/
├── AGENTS.md
├── docs/                 # Session constitution + stack + CWS
├── product/
├── scripts/              # build-agent, build-sheet, build-design, pack-unpacked
├── src/
│   ├── sidepanel.*       # UI + workspaceRpc
│   ├── background.js     # sheet_host RPC + tab registry + session tab groups
│   ├── preview/sheet.*   # Univer live grid (OSS-full vendor, gitignored)
│   ├── preview/docs.*    # Univer Docs OSS tab
│   ├── preview/design.*    # tldraw Design/Slides
│   ├── preview/site.*      # website page (HTML SoT)
│   ├── preview/workLock.* / officeShortcuts.* / officeHelp.*  # in-flight lock + keys
│   ├── preview/artifactPreview.* # generic HTML/PDF view — not a layout editor
│   ├── offscreen/        # SessionWorkspaceService host
│   ├── sandbox/          # QuickJS guest page
│   ├── content_script.js # selection + user export/screenshot
│   └── agent/
│       ├── llm.js / provider
│       └── vnext/
│           ├── runSession.product.js
│           ├── sessionWorkspace/
│           ├── service/sessionWorkspaceService.js
│           ├── skills/<id>/
│           ├── primitives/
│           └── adapters/
├── tests/session-workspace/
├── tests/workspace/
└── artifacts/unpacked/   # pack output (gitignored)
```

---

## Product rules

1. Extension-only install.  
2. Paw OFF = normal browsing; Paw ON = selection channel.  
3. Capture ≠ interpretation; late-bind after prompt.  
4. Every user message → **sendMessage**.  
5. No silent expansion outside selection without intent.  
6. Model-generated code: sandbox + QuickJS only (no raw `chrome.*` / live DOM).  
7. Deliverables = real `/artifacts` bytes; claimed formats must match magic/container.  
8. Abort cancels in-flight model/tools/code.  
9. Durable: **IDB metadata + OPFS blobs** via `DurableSessionWorkspaceStore`.  
10. BYOK OpenAI-compatible HTTPS (`createPageWandLanguageModel`).  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Player-YN/PawWork_ZhuaZhua](https://github.com/Player-YN/PawWork_ZhuaZhua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
