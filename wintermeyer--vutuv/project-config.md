---
trigger: always_on
description: This is an upgrade project to bring vutuv, a legacy Phoenix application, up to the latest Elixir and Phoenix Framework.
---

This is an upgrade project to bring vutuv, a legacy Phoenix application, up to the latest Elixir and Phoenix Framework.

vutuv is mostly a classic Phoenix **controller + HTML module + `.html.heex` template** app (Phoenix 1.8 `*_html.ex` modules with `embed_templates`, living in `lib/vutuv_web/views/*` — there is no `phoenix_view` dependency), and **LiveView is being adopted incrementally** on top of it. Most pages are still controller + template; do not rewrite a controller page as a LiveView unless a task explicitly asks. There is still no `core_components.ex`.

What is already LiveView (the real-time shell, see `docs/architecture/realtime.md`): the app shell `VutuvWeb.ShellLive` (top bar + mobile bottom tab bar with live unread badges) is embedded in the shared `app` layout via `live_render` and shows on every page; the **Messages** (`/messages`) and **Notifications** (`/notifications`) pages are LiveViews under a `live_session`; the **profile** (`/:slug`, `VutuvWeb.UserProfileLive`) is a LiveView embedded by its controller via `live_render` (the controller keeps owning agent-format negotiation, so the `.md`/`.txt`/`.json`/`.xml`/`.vcf` siblings are untouched) — every state-changing control on it is reload-free (follow pill, the header card's bookmark/like glyph toggles, ⋯-menu mute/block, list follow buttons, tag endorsements) and its counts/tags update live over PubSub; the **post permalink's conversation** (`VutuvWeb.PostLive.Thread`, embedded the same way) renders a long thread as a window around the permalinked post whose "Show earlier / more" expanders load the rest over the socket (`Vutuv.Posts.thread_window/3`; agent formats keep the whole capped thread); real-time updates flow over `Vutuv.Activity` (PubSub on `"user:<id>"`) and `VutuvWeb.Presence`. The layout is split into `root.html.heex` (document shell) + `app.html.heex` (chrome), shared by both dead and live pages. When you touch the shell, layouts, or real-time features, LiveView is expected. The email chokepoint and CSRF/PIN rules below still apply unchanged.

Framework conventions (Elixir, Ecto, Phoenix, HEEx, LiveView, assets) and the **"Direction A" visual design system** (`.claude/rules/design.md`) live in `.claude/rules/` and load automatically only when you edit a matching file, so they stay out of context the rest of the time. Use that design rule, the `VutuvWeb.UI` components, and the `assets/css/components.css` reskin rather than inventing new styles; a PostToolUse hook reminds you to keep `design.md` in sync when you change the design sources.

## Project guidelines

- Use the `mix test` alias when you are done with all changes and fix any pending issues (it runs `ecto.create` + `ecto.migrate` first).
- **New site pages live under `/system/`, never at a new root path word.** Profiles own the URL root (`/:slug`), so every new root segment (a listing, a directory, a tool, a stats page) permanently burns a word members could otherwise claim as a handle and must be listed in `Vutuv.Accounts.ReservedSlugs`. `system` is already reserved: route new site pages as `/system/<name>` (the member directory at `/system/members` set the pattern) — no new ReservedSlugs entry needed. Before ever claiming a new root word instead (only for a genuinely member-facing top-level feature, agreed with Stefan first), check the production data for members already holding it as a username (query the dev DB, a prod copy).
- **Every id is a UUID v7 — nothing else.** All primary keys and foreign keys use `Vutuv.UUIDv7` (set once in `use VutuvWeb, :model`; never override `@primary_key`/`@foreign_key_type` per schema). Migrations default to `:binary_id` via the repo config in `config/config.exs`. Mint ids in code with `Vutuv.UUIDv7.generate/0` — **never** integer ids, **never** UUID v4, **never** `Ecto.UUID.generate/0`. Inside `fragment/1` wrap pinned id params as `type(^id, Vutuv.UUIDv7)` (fragments can't infer the type). Id order matches creation order (the timestamp is in the id), so keyset tiebreakers on `id` keep working. A regression test (`test/vutuv/schema_uuid_chokepoint_test.exs`) fails the build on drift.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wintermeyer/vutuv](https://github.com/wintermeyer/vutuv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
