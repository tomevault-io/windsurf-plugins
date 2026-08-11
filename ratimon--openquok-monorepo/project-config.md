---
trigger: always_on
description: Checklist for adding a social channel — backend provider + web composer/OAuth + docs + self-host env + public SEO landing page + agent channel SEO routes + public Photo Editor channel routes + public Best Time to Post channel routes + feature bento showcases + Extensions Hub listing tags
---


# Adding a social provider

Ship **backend + web + docs + self-host env + public channel landing page + agent channel SEO routes + public Photo Editor channel routes + public Best Time to Post channel routes + feature bento showcases + Extensions Hub listing tag** in one change. Reference implementations: **Threads** (single-step OAuth), **Instagram (Business)** (Page picker + compose settings), **Facebook Page** (Meta OAuth + link-preview settings). Public landing + bento references: **facebook**, **threads** in `publicChannelConfig.ts` and `web/src/lib/ui/templates/bento/minor-templates/`. Agent channel SEO references: **facebook** at `/agents/openclaw/facebook`. Photo Editor channel references: **facebook**, **instagram** at `/tools/photo-editor/{slug}`. Best Time to Post references: **tiktok**, **linkedin** at `/tools/best-time-to-post/{slug}`; benchmark tables in `web/src/lib/best-time-to-post/constants/benchmarkSlots.ts`. Listing tag + group reference: `backend/supabase/db/listing-tags/502_20260629_seed.sql`. Full guide: `web/src/content/docs/developer-guidelines/add-provider.md`.

## Identifier contract

Use one kebab-case slug everywhere: `provider.identifier`, DB `provider_identifier`, OAuth callback `/integration/oauth/{identifier}`, `getLaunchProviderConfig`, CLI filters, docs filenames, **`publicChannelConfig` `slug` / `platformId`**, **`listing_tags.slug`**, `/channels/{slug}`, **`/tools/best-time-to-post/{slug}`**, **`/tools/photo-editor/{slug}`**, and **`/agents/{agentSlug}/{slug}`** (agent channel SEO). Do not fork slugs between layers. When `platformId` differs from the marketing slug (rare), still key **`benchmarkSlots.ts` `PLATFORM_WINDOWS`** by every identifier the calculator can pass (`platformSlug` from channel config / platform select).

## Backend (required)

1. **`SocialProvider` class** — `backend/integrations/providers/{id}/` implementing `social.integrations.interface.ts`: OAuth (`generateAuthUrl`, `authenticate`), `post`, `maxLength`, scopes. Split publish logic into helpers (e.g. `*GraphPublish.ts`) when non-trivial.
2. **Register** — add `new YourProvider()` in `backend/integrations/integrationManager.ts` (no new REST routes).
3. **Config** — secrets only via `config.integrations.*` in `GlobalConfig.ts` + `.env.development.example`. Also add the same empty keys to **`infra/self-host/.env.example`** (Social provider apps section) so self-host operators get the vars. Redirect URI: `oauthFrontendOrigin()` + `oauthFrontendSocialCallbackPath(identifier)`.
4. **Between-steps OAuth** — when `isBetweenSteps: true`: implement `pages()` + `fetchPageInformation()`; extend `IntegrationConnectionService.saveProviderPageForOrganization` / `preservesUserTokenForRefresh` if user token must stay in `refresh_token` (Meta Page pattern).
5. **Provider settings at publish** — read from `postDetails.settings.providerSettings`. Accept **flat CLI keys** and **nested web buckets** (e.g. `providerSettings.url` and `providerSettings.facebook.url`). Export a resolver + unit tests beside publish helpers.
6. **Tests** — unit tests for OAuth edge cases, publish payload shaping, and connection save when behavior differs.

## Extensions Hub listing tags (required for social channels)

The public **Extensions Hub** (`/extensions`) filters skills and MCP listings by **tag** and **tag group**. Each social channel needs a matching `listing_tags` row and group associations so hub filters stay aligned with shipped providers.

Follow **backend-migrations-naming** (`listing-tags_<YYYYMMDD>_seed.sql` under `backend/supabase/db/listing-tags/`). Re-aggregate migrations after seed changes.

| Artifact | Path / action |
| --- | --- |
| Channel tag row | New `INSERT` in `backend/supabase/db/listing-tags/501_*.sql` (or a later `501`-tier seed if `501` already shipped) |
| Group associations | `backend/supabase/db/listing-tags/502_*.sql` — append rows to `listing_tag_groups_listing_tags_association` with slug comments (see existing file) |
| Slug + name | `slug` = `provider.identifier`; `name` = human label (e.g. `Facebook`, `X`) |
| Description | One neutral sentence on what the channel integration covers (no third-party attribution) |
| Stable UUID | New `d5f7c000-0000-4000-a000-…` id; never reuse or reassign ids |

### Tag groups (social channels only)

A channel tag usually belongs to **Social platforms** plus one or more **content-type** groups. Tags may belong to multiple groups. Group ids and membership rules live in `502_*.sql` — keep the overview comment block there up to date.

| Group | When to add the channel tag |
| --- | --- |
| **Social platforms** | **Always** — every social channel tag |
| **Videos** | Video-first publish (e.g. YouTube, TikTok) |
| **Photos** | Image-first feed workflows (e.g. Instagram, Facebook Page photos) |
| **Text** | Text and microblog channels (e.g. Threads, LinkedIn, X) |

Agent / MCP tags (OpenClaw, Cursor, Codex, …) are **not** part of this checklist — only add those when shipping agent or MCP catalog entries, not when adding a social provider.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
