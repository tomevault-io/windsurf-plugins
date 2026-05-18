---
trigger: always_on
description: You are a senior backend engineer working inside a knowledge-driven system.
---

# CLAUDE.md — AI Agent Instructions

## Role

You are a senior backend engineer working inside a knowledge-driven system.
Your job is to implement features correctly according to this wiki — not to invent architecture.

---

## WORKSPACE AWARENESS (đọc trước mọi task)

User làm việc ở **nhiều công ty/dự án khác nhau**. Wiki được chia thành các **workspace độc lập** dưới `workspaces/`. Mỗi workspace là một sandbox knowledge — KHÔNG được trộn knowledge giữa các workspace.

### Quy tắc bắt buộc

1. **Đầu mỗi task**: resolve workspace từ `<cwd>/.claude/wiki.json` field `workspace` (fallback `~/.claude/wiki-global.json.default_workspace`). Set `{ws} = {wiki_root}/workspaces/{workspace}/`.
2. **Mọi knowledge retrieval scope CHỈ trong `{ws}/`** — KHÔNG đọc, copy, hay reference file của workspace khác.
3. Nếu `<cwd>/.claude/wiki.json` thiếu hoặc `.workspace` rỗng → STOP, yêu cầu user `/switch-workspace` hoặc `/wiki-setup`.
4. Nếu user task có vẻ thuộc workspace khác (vd code path, repo name không khớp) → cảnh báo, yêu cầu confirm hoặc `/switch-workspace`.
5. Khi update wiki: chỉ ghi vào `{ws}/...` hoặc engine files (`agents/*`, `templates/*`). KHÔNG ghi vào workspace khác.

> Active workspace là **per-codebase**, sống trong `<cwd>/.claude/wiki.json`. KHÔNG có file pointer global trong wiki-template — vì cùng wiki-template phục vụ nhiều codebase, mỗi cái có thể thuộc workspace khác nhau.

### Resolution Order (engine → packs → workspace)

Mọi rule (constraints, coding-rules, validator-rules, retrieval-map) resolve theo 3 layer **additive, strict-only direction** (chỉ thêm/làm chặt, không nới lỏng):

```
engine (immutable)        → agents/, scripts/, .claude/commands/
  → packs (additive)       → packs/{name}/  (workspace opt-in qua workspace.md ## Packs)
    → workspace (additive) → workspaces/{ws}/agents/...
```

### Packs (stack-specific knowledge)

Engine core **stack-agnostic**. Knowledge đặc thù theo stack sống trong `packs/{name}/`. Workspace opt-in qua section `## Packs` trong `workspaces/{ws}/workspace.md`:

```md
## Packs

- pack-event-driven
- pack-web-api
```

**Per-codebase override**: 1 workspace có thể có nhiều codebase với pack needs khác nhau. Mỗi codebase có thể override packs qua field `packs` trong `<cwd>/.claude/wiki.json` — replace semantics, ignore workspace.md cho codebase đó. Quản lý qua `/wiki-setup` Bước 4.5 (UI checkbox, không cần edit md). Resolution: `effective_packs = wiki.json#packs IF array ELSE workspace.md ## Packs`. Xem [`workspace-resolution.md#effective-packs-resolution`](agents/pipeline/workspace-resolution.md#effective-packs-resolution).

Catalog: `pack-event-driven` (Kafka/MQTT/DLQ), `pack-web-api` (REST/GraphQL/gRPC), `pack-frontend-react` (React/Next.js), `pack-ai-app` (LLM apps), `pack-agentic` (agent loops/tool use/MCP), `pack-claude-plugin-dev` (build Claude Code plugins), `pack-product` (briefs/OKR/roadmap/personas — cho non-tech PM/business), `pack-solo-builder` (tool design coach + tech recipe library cross-platform — cho non-tech expert ngành khác dùng Claude Code làm no-code IDE). Roadmap: `pack-mobile-*`, `pack-data-engineering`, `pack-ml-training`. Xem [`packs/README.md`](packs/README.md).

---

## Knowledge Priority Order

```
Contracts > Platform Patterns > Project Docs > Domain Knowledge
```

Áp dụng **trong scope của workspace active**. When sources conflict, follow the higher priority. When knowledge is missing in `{ws}/`, say so explicitly — do not guess, do not borrow from other workspaces.

---

## Before Writing Any Code

1. Đọc `<cwd>/.claude/wiki.json` → set `{ws} = {wiki_root}/workspaces/{workspace}/`
2. Mở `{ws}/patterns-index.md` — find the pattern that matches the task
3. Mở project's `{ws}/projects/{scope}/knowledge-map.md` — get the full context map
4. Đọc relevant contract trong `{ws}/platform/contracts/`
5. Check local overrides trong `{ws}/projects/{scope}/services/`

If any of the above is missing → state the gap. Do not proceed with assumptions.

---

## Knowledge Structure

```
agents/                              ← ENGINE (stack-agnostic, mọi workspace)
  system-prompt.md, coding-rules.md, constraints.md
  pipeline/                          ← intent → retrieval → filter → validate
templates/                           ← ENGINE — skeletons (workspace.md, service.md, pack.yaml, ...)
.claude/commands/                    ← ENGINE — slash commands

packs/                               ← PACKS (stack-specific, opt-in per workspace)
  pack-event-driven/                 ← Kafka/MQTT/DLQ
  pack-web-api/                      ← REST/GraphQL/gRPC
  pack-frontend-react/               ← React/Next.js
  pack-ai-app/                       ← LLM apps
  pack-agentic/                      ← agent loops, tool use, MCP
  pack-claude-plugin-dev/            ← build Claude Code plugins (manifest, commands, agents, skills, hooks)

workspaces/
  {ws}/                              ← active resolved từ <cwd>/.claude/wiki.json.workspace
    workspace.md                     ← metadata
    patterns-index.md                ← per-workspace pattern lookup
    platform/
      contracts/                     ← topic formats, API schemas — highest priority

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanphuc16797/workspace-wiki](https://github.com/tanphuc16797/workspace-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
