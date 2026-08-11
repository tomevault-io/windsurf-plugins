---
trigger: always_on
description: Meta-guidance for OpenQuok agent skills under agent/skills/. Authoritative instructions live in each skill’s SKILL.md — apply when editing those skills or this rule.
---


**Sources of truth:** each installable skill’s `SKILL.md` under `agent/skills/<slug>/`. Edit that file when changing what agents should do; keep this rule for Cursor-only best practices and cross-skill layout.

## Skill families

| Kind | Example | Layout | Role |
|------|---------|--------|------|
| **Core CLI skill** | `openquok-core` | `SKILL.md` + `resources/` | Auth, media Rule 2, channel recipes, `openquok` workflows |
| **Sibling pipeline skill** | `openquok-tiktok-slideshow` | `SKILL.md` + `scripts/` + `references/` | Domain pipeline (research → generate → post); **requires** `openquok-core` / `openquok` on PATH; never replaces core |

Sibling skills may depend on core recipes (upload → `{id,path}`, TikTok photo-carousel, `SELF_ONLY` drafts) by instructing the agent to follow `openquok-core` — do not duplicate long CLI matrices into the sibling `SKILL.md`.

Publish each slug separately to ClawHub (`agent/CLAWHUB.md`). Skills with `scripts/` must document **Copy / `--copy`** install so scripts land as real files.

## Skill layout — openquok-core

- **`SKILL.md`** — Hard rules, auth, core workflow, pitfalls, quick reference. Keep it focused; link out instead of duplicating long examples. The **Channels (Meta)** table indexes per-platform files and summarizes user intents — do not paste feature matrices or long bash into `SKILL.md`.
- **`resources/*.md`** — Deeper material agents load on demand:
  - `command-reference.md` — CLI commands, flags, env vars (not `agent/README.md`; skill install path has no sibling README).
  - `provider-settings.md` — shared publish-time mechanics: `--settings` vs `--providerSettingsByIntegrationId`, merge order, flat vs nested API buckets, multi-`-c` behavior.
  - `patterns.md` — extended workflows (campaign JSON, batching, retries).
  - `{identifier}-examples.md` — per-channel agent docs (see **Per-channel example file** below). Today: `threads`, `facebook`, `instagram-standalone`, `instagram-business`.
  - `threads-publish.md` — server publish behavior summarized in markdown (never link to `.ts` / backend paths from the skill).
- **`SKILL.md` tone** — imperative, short tables and bullets; no tutorial prose; shell-safety rules for untrusted input; defer command lists and long bash to `resources/`.

## Skill layout — sibling skills (`scripts/` + `references/`)

Use this pattern for pipeline skills that ship runnable helpers (e.g. `openquok-tiktok-slideshow`):

- **`SKILL.md`** — Phases, prerequisites, install (Copy required), config shape pointers, and how to invoke scripts. Keep lean; link into `references/` for deep playbooks.
- **`scripts/`** — Node (or shell) helpers the agent runs with `node scripts/<name>.js`. Prefer CommonJS or whatever the skill’s local `package.json` declares. Do not assume npm deps beyond `engines` unless the skill documents a real install step on its homepage.
- **`references/`** — On-demand docs and scaffolds agents load when needed (slide structure, research notes, character-lock templates, JSON templates). Prefer neutral first-party templates — no personal asset paths or third-party persona dumps in-repo.
- **Compatibility** — State clearly: does not replace `openquok-core`; requires `openquok` (+ `node`) on PATH; media and post create go through the CLI.
- **Workspace config** — Onboarding may scaffold files **outside** the skill install (user workspace). Document paths in `SKILL.md`; do not write secrets into the skill bundle.

When adding a new sibling skill, also update ClawHub scripts in `agent/package.json`, `agent/CLAWHUB.md`, and the Extensions Hub listing / docs page as applicable.

## Per-channel example file (`resources/{identifier}-examples.md`)

Each live Meta (or future) channel the CLI can post to should have one example file matching `provider.identifier`. Structure every file the same way so agents can map **user intent → recipe**:

1. **Resolve integration** — `integrations:list` + `integrations:settings` one-liner at the top.
2. **Supported features** — table: Feature | Supported | Notes — **only shipped behavior** (same bar as `publicChannelConfig` FAQ/bento, but agent-actionable, not marketing copy).
3. **Agent tasks** — table: “User wants to…” | “Do this” — link to anchored sections below (e.g. link preview, Reel, reply chain).
4. **Provider settings** — table of publish keys (`--settings` / `providerSettingsByIntegrationId`), with precedence rules (e.g. Facebook `url` ignored when media attached).
5. **Recipes** — short `bash` blocks per task; use `<integration-id>` placeholders; link to `provider-settings.md` once at the top.

When adding or changing publish behavior for a channel, update **backend resolver + this file** together. Do not lift `publicChannelConfig.ts` hero/feature prose into the skill — distill facts into the tables above.

## CLI-only scope

Skill content is for agents using `openquok` and the public API.

- **Do** document flat API keys the backend accepts on create (`post_type`, `url`, `is_trial_reel`, …) and nested **orchestrator buckets** in `providerSettingsByIntegrationId` (`threads.replies`, `threads.internalEngagementPlug`, `instagram.replies`, `facebook.url` when nested).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
