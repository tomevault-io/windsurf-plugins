---
trigger: always_on
description: This is a Phoenix web application governed by ControlKeel.
---

This is a Phoenix web application governed by ControlKeel.

## ControlKeel Governance

Before shell, code, config, deploy, or broad cleanup work:

1. Call `ck_context` for mission, budget, findings, proof, workspace, and resume state.
2. Invoke the `controlkeel-governance` skill.
3. Call `ck_budget` before expensive, multi-phase, or delegated work.
4. Call `ck_validate` before shell commands, file writes, generated code, config changes, or deploy actions.
5. Use `ck_review_submit` and wait for `ck_review_status` approval before broad mutations or risky deletion.
6. Record durable decisions with `ck_memory_record`.
7. Record discovered issues with `ck_finding`.
8. Use `ck_route` before delegation.

Fresh sessions must reacquire CK state even if a conversation summary exists. If MCP is unavailable, manually check risk, findings, budget, and security implications before changing files, then leave clear session notes.

Keep this root file lean: only project-specific governance, commands, and critical gotchas belong here. Put reusable agent guidance in skills or focused docs.

## Issue and PR Quality

- Use evidence-based issue reporting: command run, expected behavior, actual behavior, exact error/log
- Avoid over-confident root cause analysis without supporting evidence
- When reviewing AI-generated contributions, use `ck_validate` with `source_type: "issue"` or `source_type: "pull_request"`
- Prefer invariant enforcement over local workarounds for bad states
- Use `deslop` skill to clean AI-generated issue/PR text slop

## Planning and Architecture

- Use structural planning: types/interfaces + call stacks + boundaries over prose descriptions
- Include call graphs to show what's allowed to talk to what (boundary enforcement)
- Apply targeted review loops (revise specific sections, don't rewrite everything)
- Let models correct their own mistakes using validation tools in agentic loops

## Project Commands

- Run `mix precommit` after code changes and fix pending issues.
- Use `Req` for HTTP; avoid `:httpoison`, `:tesla`, and `:httpc`.
- Use `apply_patch` for manual edits when available. Do not use shell writes for file edits.
- Read `mix help <task>` before unfamiliar Mix tasks.
- Prefer targeted tests first: `mix test test/path/to_test.exs` or `mix test --failed`.
- Avoid `mix deps.clean --all` unless there is a specific dependency corruption reason.

## Phoenix v1.8

- All layouts are **framework layouts** owned by `ControlKeelWeb.Layouts` (`components/layouts.ex`, `embed_templates "layouts/*"`): `:root`, `:public`, `:dashboard`, `:observability`, `:observability_session`. LiveView/controller templates render only their content; the framework wraps them — do NOT add per-page layout wrappers.
- Set the layout per `live_session` (`layout: {ControlKeelWeb.Layouts, :dashboard}`) or per controller (`plug :put_layout, html: {ControlKeelWeb.Layouts, :public}`). The root layout is set via `put_root_layout` in the browser pipeline.
- Framework layouts share the page's render context, so `@flash`, `@current_user`, `@current_membership`, and `@inner_content` are available with **no forwarding**. `@current_path` (subnav/tab highlighting) is set by the `ControlKeelWeb.NavHighlight` `on_mount` hook on `:observability` / `:observability_session`.
- Reusable chrome (`<.sidebar>`, `<.flash_group>`) lives as function components in the `Layouts` module. `<.flash_group>` belongs there, called by the framework layouts.
- See `docs/framework-layout-migration.md` for the migration rationale.
- Use `<.icon name="hero-x-mark" class="w-5 h-5"/>`; do not call Heroicons modules directly.
- Use imported `<.input>` for form inputs. If overriding `class`, fully style the input because defaults are not inherited.
- Router scopes already include aliases; do not add duplicate route aliases.
- Do not use `Phoenix.View`.

## Elixir and Ecto

- Use `Enum.at/2` for list index access.
- Rebind block results outside the block.
- Do not nest multiple modules in one file.
- Access struct fields with `struct.field`, not map syntax.
- Use stdlib `Time`, `Date`, and `DateTime`; only use extra parsing deps already in the project.
- Never use `String.to_atom/1` on user input.
- Predicate functions end in `?`; reserve `is_` names for guards.
- Name OTP child specs, for example `{DynamicSupervisor, name: MyApp.Sup}`.
- Use `Task.async_stream/3` with back-pressure for concurrent enumeration.
- Preload associations used in templates.
- Use `:string` for schema text fields.
- `validate_number/2` has no `:allow_nil` option.
- Use `Ecto.Changeset.get_field/2` for changeset fields.
- Set programmatic fields like `user_id` explicitly, not through `cast`.
- Create migrations with `mix ecto.gen.migration descriptive_name`.

## HEEx and LiveView

- Use `~H` and `.html.heex`, never `~E`.
- Use `Phoenix.Component.form/1`, `inputs_for/1`, and `to_form/2`; do not use `Phoenix.HTML.form_for` or `<.form let={f}>`.
- Give every form a unique DOM id.
- Import app-wide helpers through the `html_helpers` block.
- Use `cond` or `case`; Elixir has no `else if`.
- Use `phx-no-curly-interpolation` for literal braces in `<code>` and `<pre>`.
- Use list syntax for class attrs and wrap inline `if` expressions.
- Use `<%= for %>` for template loops, not `Enum.each`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aryaminus/controlkeel](https://github.com/aryaminus/controlkeel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
