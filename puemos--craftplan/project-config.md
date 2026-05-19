---
trigger: always_on
description: - Craftplan is a self-hosted inventory & manufacturing software for micro-business, covering catalog, inventory, orders, CRM, and planner flows.
---

# Craftplan Agent Guide

## Project Snapshot
- Craftplan is a self-hosted inventory & manufacturing software for micro-business, covering catalog, inventory, orders, CRM, and planner flows.
- Core stack: Elixir 1.18+, Ash Framework for domain modeling, AshPostgres for persistence, Phoenix LiveView + TailwindCSS for UI.
- Product principles: keep the operate → make → stock loop simple, favor printable artifacts (planner, labels, invoices), and ship durable primitives over heavy configuration.

## Repo Layout Highlights
- `lib/craftplan/`: Ash domains by context (`catalog`, `inventory`, `orders`, `settings`, `crm`, `csv`, `types`).
- `lib/craftplan_web/`: Phoenix boundary (LiveViews in `live/`, shared components in `components/`, controllers for public endpoints).
- `priv/repo/`: Database migrations and `seeds.exs`; keep `priv/resource_snapshots/` aligned with Ash migrations.
- `assets/`: Frontend assets (requires `npm install --prefix assets` before builds).
- `test/`: Domain tests under `craftplan/`, LiveView integration under `craftplan_web/`, shared helpers in `test/support`.

## Working Locally
- First setup: `mix setup` (runs deps.get, `ash.setup`, asset install/build, seeds).
- Start server: `mix phx.server`; demo data seeded from `priv/repo/seeds.exs`.
- Reset data: `mix ash.reset` for extension-aware rebuilds or `mix ash_postgres.drop && mix setup` after major schema changes.
- Assets: run `npm install --prefix assets` once; `mix assets.deploy` creates production bundles.
- Environment: `.envrc`/`config/runtime.exs` hold runtime secrets; mirror updates in `docker-compose.yml` when containerizing.

## Ash Patterns & Expectations
- Express business rules in `Ash.Resource` (attributes, relationships, `changes`, `validations`) and avoid imperative LiveView logic.
- Use `manage_relationship` helpers for nested updates; avoid manual Repo access in the web layer.
- Preload data with `Ash.Query.load/2` or `Ash.load/3` before pushing to LiveViews to keep UI resource agnostic.
- After schema changes run `mix ash_postgres.generate_migrations` to update migrations *and* `priv/resource_snapshots/`; commit both.
- Create explicit read/update actions for LiveViews instead of ad-hoc queries; leverage `Ash.Flow` or `Ash.Changeset.for_action` for multi-step operations.

## Phoenix & LiveView Notes
- Use `to_form` assigns with `<.form>`; never access raw changesets in HEEx templates.
- Prefer LiveView streams (`stream/3`) for long-running collections and track counts with dedicated assigns.
- Extend shared UI in `CraftplanWeb.Components` instead of duplicating markup.
- Place JS hooks in `assets/js` and wire via `app.js`; never embed `<script>` tags in HEEx.
- Check router scopes for implicit aliases when adding LiveView routes (e.g., `/manage/orders` inside the browser scope).

## Testing & QA
- Default run: `mix test` (alias runs `ash.setup --quiet` first); narrow with `mix test path/to/file.exs:line`.
- LiveView specs use `Phoenix.LiveViewTest` helpers (`live/2`, `element/2`, `render_change/2`, `render_submit/2`) and assert against DOM IDs defined in templates.
- Reuse fixtures/factories in `test/support`; avoid raw SQL fixtures.
- Run `mix format` before commits; formatter covers Elixir and HEEx.

## Reference & Tooling
- For docs, use `mix usage_rules.docs <Module>` and `mix usage_rules.search_docs "query"`.
- Visual aids: `mix ash.generate_resource_diagrams`, `mix ash.generate_policy_charts`.
- Keep `PLAN.md` milestone checkboxes updated as features land.

## Git Rules
- Commit message format: `<type>(<scope>): <subject>`

## Usage Rules Appendix

<!-- usage-rules-start -->
<!-- usage-rules-header -->
# Usage Rules

**IMPORTANT**: Consult these usage rules early and often when working with the packages listed below.
Before attempting to use any of these packages or to discover if you should use them, review their
usage rules to understand the correct patterns, conventions, and best practices.
<!-- usage-rules-header-end -->

<!-- usage_rules-start -->
## usage_rules usage
_A dev tool for Elixir projects to gather LLM usage rules from dependencies_

## Using Usage Rules

Many packages have usage rules, which you should *thoroughly* consult before taking any
action. These usage rules contain guidelines and rules *directly from the package authors*.
They are your best source of knowledge for making decisions.

## Modules & functions in the current app and dependencies

When looking for docs for modules & functions that are dependencies of the current project,
or for Elixir itself, use `mix usage_rules.docs`

```
# Search a whole module
mix usage_rules.docs Enum

# Search a specific function
mix usage_rules.docs Enum.zip

# Search a specific function & arity
mix usage_rules.docs Enum.zip/1
```


## Searching Documentation

You should also consult the documentation of any tools you are using, early and often. The best
way to accomplish this is to use the `usage_rules.search_docs` mix task. Once you have
found what you are looking for, use the links in the search results to get more detail. For example:

```
# Search docs for all packages in the current application, including Elixir
mix usage_rules.search_docs Enum.zip


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puemos/craftplan](https://github.com/puemos/craftplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
