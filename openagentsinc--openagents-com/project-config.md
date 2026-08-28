---
trigger: always_on
description: This is a web application written using the Phoenix web framework.
---

This is a web application written using the Phoenix web framework.

## Vocabulary

Read `docs/taxonomy.md` before writing docs, commit messages, or product copy.
It defines what each word means here — forge versus GitHub, push versus
deploy, computer versus machine, which receipt, which module — and the naming
rules that keep them straight.

## Communication style

All text in this repo — docs, README, `AGENTS.md`, commit messages, and agent responses — follows the [Google Developer Documentation Style Guide](https://developers.google.com/style). When writing or reviewing text, read the `google-developer-style` skill at `.agents/skills/google-developer-style/SKILL.md`.

- Write in active voice and address the reader as `you`.
- Use sentence case for all headings and titles.
- Use code font for code, filenames, class names, HTTP status codes, and placeholders.
- Use bold for UI elements.
- Use numbered lists for procedures and bulleted lists for unrelated items.
- Avoid jargon, buzzwords, metaphors, exclamation marks, and phrases like `simply` or `just`.
- Avoid `please` in instructions.

## Git remotes

Push to the forge, never to GitHub:

```sh
git push openagents HEAD:main
```

The `openagents` remote is the forge at `openagents.com`, which records every
push in the durable WAL and serves it. The `origin` remote is the GitHub
mirror; pushing to it directly leaves the forge behind a mirror it does not
know about, and nothing reports the divergence until a clone disagrees with
the site. Production mirrors to GitHub with a force push of every ref, so a
direct GitHub push is not merged later — it is overwritten. GitHub
holds what the forge last mirrored there. `ops/ci/push-remote-check.sh` refuses
a non-forge push, and `mix precommit` installs it into the clone you are
working in, so running precommit before you push is enough. To install it by
hand:

```sh
sh ops/dev/install-push-guard.sh
```

One install covers every worktree of that clone. Where `core.hooksPath` points
at `.githooks`, its `pre-push` runs the same guard ahead of the release gate.
See `INVARIANTS.md`, REPOSITORY-002.

## Project guidelines

- Use `mix precommit` alias when you are done with all changes and fix any pending issues
- Use the already included and available `:req` (`Req`) library for HTTP requests, **avoid** `:httpoison`, `:tesla`, and `:httpc`. Req is included by default and is the preferred HTTP client for Phoenix apps

### Phoenix v1.8 guidelines

- **Always** begin your LiveView templates with `<Layouts.app flash={@flash} ...>` which wraps all inner content
- The `MyAppWeb.Layouts` module is aliased in the `my_app_web.ex` file, so you can use it without needing to alias it again
- Anytime you run into errors with no `current_scope` assign:
  - You failed to follow the Authenticated Routes guidelines, or you failed to pass `current_scope` to `<Layouts.app>`
  - **Always** fix the `current_scope` error by moving your routes to the proper `live_session` and ensure you pass `current_scope` as needed
- Phoenix v1.8 moved the `<.flash_group>` component to the `Layouts` module. You are **forbidden** from calling `<.flash_group>` outside of the `layouts.ex` module
- Render icons only through `OpenAgentsWeb.UI.icon/1`. Prefer the vendored Apps SDK set. Use a `hero-*` fallback only when `docs/ICONS.md` records why the preferred set has no suitable glyph
- **Always** use the imported `OpenAgentsWeb.UI.input/1` component for form inputs. It accepts `Phoenix.HTML.FormField` values and unwrapped raw controls
- If you override the default input classes (`<.input class="myclass px-2 py-1 rounded-lg">)`) class with your own values, no default classes are inherited, so your
custom classes must fully style the input

### JS and CSS guidelines

- **Use Tailwind CSS classes and custom CSS rules** to create polished, responsive, and visually stunning interfaces.
- Tailwindcss v4 **no longer needs a tailwind.config.js** and uses a new import syntax in `app.css`:

      @import "tailwindcss" source(none);
      @source "../css";
      @source "../js";
      @source "../../lib/my_app_web";

- **Always use and maintain this import syntax** in the app.css file for projects generated with `phx.new`
- **Never** use `@apply` when writing raw css
- **There is exactly one component system: vendored Basecoat plus OpenAgents style pack.** Basecoat lives in `assets/vendor/basecoat/components/` and carries structure (display, padding, min-height); `assets/css/openagents.css` carries OpenAgents' identity (motion tokens, radius scale, color, the notched variant, corner frames) and must stay the last import so its declarations win.
  - **Reach for `OpenAgentsWeb.UI` first.** It wraps that CSS in 22 ready primitives — `button/1`, `card/1`, `badge/1`, `alert/1`, `input/1`, `textarea/1`, `label/1`, `field/1`, `header/1`, `table/1`, `list/1`, `avatar/1`, `menu/1`, `empty/1`, `kbd/1`, and the rest. Fall back to hand-written classes only when no primitive covers the shape.
  - **Variants are data attributes, not classes.** A control is `class="btn"` plus `data-variant="primary"` / `data-size="sm"` / `data-tone="danger"`. Never invent `btn-primary`-style variant classes; they defeat the whole point of the split.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenAgentsInc/openagents.com](https://github.com/OpenAgentsInc/openagents.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
