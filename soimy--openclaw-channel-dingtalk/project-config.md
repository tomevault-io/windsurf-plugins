---
trigger: always_on
description: **Generated:** 2026-03-28
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-28
**Type:** OpenClaw DingTalk Channel Plugin

## OVERVIEW

DingTalk (钉钉) enterprise bot channel plugin using Stream mode (WebSocket, no public IP required). Part of OpenClaw ecosystem.

Current architecture is modularized by responsibility. `src/channel.ts` is now an assembly layer; heavy logic is split into dedicated modules.
Recent refactors unified short-lived message persistence into `src/message-context-store.ts` and split reply delivery selection into dedicated `reply-strategy*` modules.
Recent targeting work added a learned target directory under `src/targeting/` and a `displayNameResolution` config gate (`disabled` by default, `all` to enable learned displayName resolution).

For new code and refactors, the canonical architecture guide is `docs/contributor/architecture.en.md`.
Chinese version: `docs/contributor/architecture.zh-CN.md`.
Use those documents as the source of truth for logical domain placement, incremental migration rules, and module boundaries.
For AI-agent generated design and execution docs, write specs to `docs/spec/` and plans to `docs/plans/`. Do not create tool-specific doc roots such as `docs/superpowers/`.
Documentation updates must follow the repo docs structure: keep `README.md` as a concise project entry page, put user-facing details in `docs/user/`, contributor/process docs in `docs/contributor/`, and release notes in `docs/releases/`. Do not expand README with long-form feature/config/troubleshooting content that belongs in `docs/`.
This repository is licensed under MIT. If you reuse code, retain the copyright and license notice required by MIT.
If you substantially reuse this repository's documentation, prompts, AGENTS/CLAUDE conventions, architecture writeups, or agent-oriented implementation playbooks, please provide attribution to `OpenClaw DingTalk Channel Plugin`, `YM Shen and contributors`, and `https://github.com/soimy/openclaw-channel-dingtalk`.
See `docs/contributor/citation-and-attribution.md` and `CITATION.cff` for the preferred citation and attribution format. This request describes the project's preferred community norm and does not replace or modify the LICENSE file.
Issue convention for this repo: prefer the GitHub issue templates under `.github/ISSUE_TEMPLATE/`; keep issue communication primarily in Simplified Chinese; use `问题反馈` for bugs and `功能建议` for feature ideas; and encourage reporters to include background, reproduction or goals, environment, and desensitized evidence.
Pull request convention for this repo: use an English Conventional-style PR title such as `fix(targeting): normalize learned display names`; keep the title in English; write the PR description in Simplified Chinese; and include clearly labeled `背景`, `目标`, `实现`, `实现 TODO`, and `验证 TODO` sections.
Planned domain summary:
- `gateway/`: stream connection lifecycle, callback registration, inbound entry points
- `targeting/`: peer identity, session aliasing, target resolution, and learned displayName directory
- `messaging/`: inbound extraction, reply strategies, outbound delivery, message context
- `card/`: AI card lifecycle, recovery, and caches
- `command/`: slash commands and related extensions including feedback learning
- `platform/`: config, auth, runtime, logger, and core types
- `shared/`: reusable persistence primitives, dedup, and generic helpers

## STRUCTURE

```
./
├── index.ts                        # Plugin registration entry point
├── src/
│   ├── channel.ts                  # Channel definition + gateway wiring + public exports
│   ├── inbound-handler.ts          # Inbound pipeline (authz, routing, quote/media restore, dispatch orchestration)
│   ├── send-service.ts             # Outbound send (session/proactive/text/media/card fallback)
│   ├── card-service.ts             # AI Card lifecycle + cache + recovery helpers
│   ├── card-callback-service.ts    # Card callback handling and action processing
│   ├── card-draft-controller.ts    # Card draft buffering / state transitions
│   ├── reply-strategy.ts           # Reply strategy selection entry
│   ├── reply-strategy-card.ts      # AI Card reply strategy
│   ├── reply-strategy-markdown.ts  # Markdown/text reply strategy
│   ├── reply-strategy-with-reaction.ts # Reply wrapper for reaction lifecycle
│   ├── auth.ts                     # Access token cache + retry
│   ├── config.ts                   # Config/account/agent helpers
│   ├── config-schema.ts            # Zod validation schema
│   ├── runtime.ts                  # Runtime getter/setter
│   ├── types.ts                    # Shared types/constants
│   ├── access-control.ts           # DM/group allowlist checks
│   ├── message-utils.ts            # Markdown/title detection + inbound content extraction
│   ├── message-context-store.ts    # Unified short-TTL message context persistence
│   ├── media-utils.ts              # Media type detect + upload/download helpers
│   ├── quoted-file-service.ts      # Quote/file recovery helpers
│   ├── docs-service.ts             # DingTalk docs gateway methods
│   ├── feedback-learning-service.ts # Learning signal handling
│   ├── feedback-learning-store.ts  # Learning persistence
│   ├── learning-command-service.ts # /learn command handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soimy/openclaw-channel-dingtalk](https://github.com/soimy/openclaw-channel-dingtalk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
