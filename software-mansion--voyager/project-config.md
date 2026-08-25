---
trigger: always_on
description: This is a web application written using the Phoenix web framework.
---

This is a web application written using the Phoenix web framework.

## Project guidelines

- Use `mix precommit` alias when you are done with all changes and fix any pending issues
- Use the already included and available `:req` (`Req`) library for HTTP requests, **avoid** `:httpoison`, `:tesla`, and `:httpc`. Req is included by default and is the preferred HTTP client for Phoenix apps

### Phoenix v1.8 guidelines

- The app shell (navbar, sidebar, statusbar) and flash are injected automatically via `live_session` layout — LiveViews render **only their own page content**, not the shell
- The `VoyagerWeb.Layouts` module is aliased in `voyager_web.ex`, so you can use it without needing to alias it again
- Phoenix v1.8 moved the `<.flash_group>` component to the `Layouts` module. You are **forbidden** from calling `<.flash_group>` outside of the `layouts.ex` module
- Icons live in `assets/css/icons/` as SVG files. `core_components.ex` provides `<.icon name="icon-x" class="size-4"/>`. **Always** use the `<.icon>` component for icons — to add a new icon drop an SVG into `assets/css/icons/` and reference it as `icon-{filename}`
- **Always** use the imported `<.input>` component for form inputs from `core_components.ex` when available. `<.input>` is imported and using it will save steps and prevent errors
- If you override the default input classes (`<.input class="myclass px-2 py-1 rounded-lg">)`) class with your own values, no default classes are inherited, so your
  custom classes must fully style the input

### JS and CSS guidelines

- **Always use DaisyUI components** (`btn`, `menu`, `navbar`, `alert`, `badge`, `card`, `modal`, `drawer`, etc.) as the primary building blocks for all UI. Supplement with Tailwind utility classes for spacing, layout, and one-off adjustments
- Active DaisyUI themes: `voyager-light` (default) and `voyager-dark` — switch via `data-theme` on `<html>`
- DaisyUI is installed as an npm package in `assets/` — use `@plugin "daisyui"` and `@plugin "daisyui/theme"` in `app.css`
- **Always** use DaisyUI semantic color tokens (`bg-primary`, `text-base-content`, `bg-base-100`, `text-success`, `bg-error`, etc.) for colors
- Tailwind v4 **no longer needs a tailwind.config.js** and uses a new import syntax in `app.css`:

      @import "tailwindcss" source(none);
      @source "../css";
      @source "../js";
      @source "../../lib/voyager_web";

- **Always use and maintain this import syntax** in `app.css`
- **Never** use `@apply` when writing raw CSS
- Out of the box **only the app.js and app.css bundles are supported**
  - You cannot reference an external vendor'd script `src` or link `href` in the layouts
  - You must import the vendor deps into app.js and app.css to use them
  - **Never write inline <script>custom js</script> tags within templates**
- Use Tailwind's built-in scale (`text-xs` … `text-3xl`, `max-w-7xl`, `p-5`, `w-16`, …) — not arbitrary `[Npx]` / `[Nrem]` values. DaisyUI has no font-size utilities; use `text-*` for typography.

### UI/UX & design guidelines

- **Produce world-class UI designs** with a focus on usability, aesthetics, and modern design principles
- Use DaisyUI as your design foundation — leverage semantic variants like `btn-primary`, `alert-info`, `badge-success`, etc.
- Implement **subtle micro-interactions** (e.g., button hover effects, and smooth transitions)
- Ensure **clean typography, spacing, and layout balance** for a refined, premium look
- Focus on **delightful details** like hover effects, loading states, and smooth page transitions

<!-- usage-rules-start -->

<!-- phoenix:elixir-start -->

## Elixir guidelines

- Elixir lists **do not support index based access via the access syntax**

  **Never do this (invalid)**:

      i = 0
      mylist = ["blue", "green"]
      mylist[i]

  Instead, **always** use `Enum.at`, pattern matching, or `List` for index based list access, ie:

      i = 0
      mylist = ["blue", "green"]
      Enum.at(mylist, i)

- Elixir variables are immutable, but can be rebound, so for block expressions like `if`, `case`, `cond`, etc
  you _must_ bind the result of the expression to a variable if you want to use it and you CANNOT rebind the result inside the expression, ie:

      # INVALID: we are rebinding inside the `if` and the result never gets assigned
      if connected?(socket) do
        socket = assign(socket, :val, val)
      end

      # VALID: we rebind the result of the `if` to a new variable
      socket =
        if connected?(socket) do
          assign(socket, :val, val)
        end

- **Never** nest multiple modules in the same file as it can cause cyclic dependencies and compilation errors
- **Never** use map access syntax (`changeset[:field]`) on structs as they do not implement the Access behaviour by default. For regular structs, you **must** access the fields directly, such as `my_struct.field` or use higher level APIs that are available on the struct if they exist, `Ecto.Changeset.get_field/2` for changesets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [software-mansion/voyager](https://github.com/software-mansion/voyager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
