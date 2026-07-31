---
trigger: always_on
description: Cabinet is an AI-first self-hosted knowledge base and startup OS. All content lives as markdown files on disk. The web UI provides WYSIWYG editing, a collapsible tree sidebar, drag-and-drop page organization, structured AI runs for tasks/jobs/heartbeats, and interactive `WebTerminal` surfaces for direct CLI sessions.
---

# CLAUDE.md — Cabinet

## What is this project?

Cabinet is an AI-first self-hosted knowledge base and startup OS. All content lives as markdown files on disk. The web UI provides WYSIWYG editing, a collapsible tree sidebar, drag-and-drop page organization, structured AI runs for tasks/jobs/heartbeats, and interactive `WebTerminal` surfaces for direct CLI sessions.

**Core philosophy:** Humans define intent. Agents do the work. The knowledge base is the shared memory between both.

## Tech Stack

- **Framework:** Next.js 16 (App Router), TypeScript
- **UI:** Tailwind CSS + shadcn/ui (base-ui based, NOT Radix — no `asChild` prop)
- **Editor:** Tiptap (ProseMirror-based) with markdown roundtrip via HTML intermediate
- **State:** Zustand (tree-store, editor-store, ai-panel-store, task-store, app-store)
- **Fonts:** Inter (sans) + JetBrains Mono (code)
- **Icons:** Lucide (no emoji in system chrome)
- **Markdown:** gray-matter (frontmatter), unified/remark (MD→HTML), turndown (HTML→MD)
- **AI providers:** Claude Code, Codex CLI, Cursor CLI, OpenCode, Copilot CLI, Grok CLI, Pi, and a generic CLI adapter — all driven through the shared adapter runtime in `src/lib/agents/`.

## Architecture

```
src/
  app/api/tree/              → GET tree structure from /data
  app/api/pages/[...path]/   → GET/PUT/POST/DELETE/PATCH pages
  app/api/upload/[...path]/  → POST file upload to page directory
  app/api/assets/[...path]/  → GET/PUT static file serving + raw file writes
  app/api/search/            → GET full-text search
  app/api/agents/conversations/ → Manual task/conversation creation + listing
  app/api/agents/providers/  → Provider, model, adapter metadata
  app/api/agents/tasks/      → Task board data
  app/api/agents/scheduler/  → Scheduler control/status
  app/api/agents/skills/     → Skill library: list/CRUD, import (github/skills.sh/local), bundle-into-cabinet, trust, scan, catalog
  app/api/git/               → Git log, diff, commit endpoints
  stores/                    → Zustand (tree, editor, ai-panel, task, app)
  components/sidebar/        → Tree navigation, drag-and-drop, context menu
  components/editor/         → Tiptap WYSIWYG + toolbar, website/PDF/CSV/office viewers
  components/editor/office/  → Read-only viewers for .docx, .xlsx, .pptx
  components/ai-panel/       → Right-side AI chat panel
  components/tasks/          → Task board + task detail panel
  components/agents/         → Agents workspace + live/result conversation views
  components/jobs/           → Jobs manager UI
  components/terminal/       → xterm.js web terminal
  components/composer/       → Shared composer + task runtime picker (supports @page, @agent, @skill mentions)
  components/skills/         → Skill library, detail page, add dialog, picker, "Skills offered" transcript footer
  components/search/         → Cmd+K search dialog
  components/layout/         → App shell, header
  lib/storage/               → Filesystem ops (path-utils, page-io, tree-builder, task-io)
  lib/markdown/              → MD↔HTML conversion
  lib/git/                   → Git service (auto-commit, history, diff)
  lib/agents/                → Adapter runtime, conversation runner, personas, providers
  lib/agents/skills/         → Multi-origin skill loader, trust gating, sync (mount/symlink), discovery scan, lock file
  lib/jobs/                  → Job scheduler (node-cron)
server/
  cabinet-daemon.ts          → Unified daemon: structured adapter runs, PTY sessions, scheduler, event bus
  pty/                       → PTY session module: ansi, claude-lifecycle, manager, types
data/                        → Content directory (KB pages, tasks, jobs)
```

## Key Rules

1. **No database** — everything is files on disk under `/data`
2. **Pages** are directories with `index.md` + assets, or standalone `.md` files. PDFs and CSVs are also first-class content types.
3. **Frontmatter** (YAML) stores metadata: title, created, modified, tags, icon, order
4. **Path traversal prevention** — all resolved paths must start with DATA_DIR
5. **shadcn/ui uses base-ui** (not Radix) — DialogTrigger, ContextMenuTrigger etc. do NOT have `asChild`
6. **Dark mode default** — theme toggle available, use `next-themes` with `attribute="class"`
7. **Auto-save** — debounced 500ms after last keystroke in editor-store
8. **AI runs use a mixed runtime model** — tasks/jobs/heartbeats default to structured adapters; terminal mode (PTY sessions) is a first-class alternative that runs inside the same daemon process via `server/pty/`. `WebTerminal` is the interactive surface for both.
9. **Terminal is a first-class runtime** — not deprecated, not an escape hatch. Terminal mode is user-selectable per task (Native / Terminal toggle in the composer) and is the direction for future terminal-native workflows (Cabinet-managed tmux-like sessions).
10. **Version restore** — users can restore any page to a previous git commit via the Version History panel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hilash/cabinet](https://github.com/hilash/cabinet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
