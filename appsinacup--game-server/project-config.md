---
trigger: always_on
description: This is a web application written using the Phoenix web framework.
---

This is a web application written using the Phoenix web framework.

## Project guidelines

- Use `mix precommit` alias when you are done with all changes and fix any pending issues
- Use the already included and available `:req` (`Req`) library for HTTP requests, **avoid** `:httpoison`, `:tesla`, and `:httpc`. Req is included by default and is the preferred HTTP client for Phoenix apps

## Repository architecture (core/web/root host)

This repo keeps `game_server_core` and `game_server_web` reusable under `apps/`, while the runnable host app now lives at the repository root.

- `apps/game_server_core`: domain code, contexts, schema, migrations, shared business logic.
- `apps/game_server_web`: reusable web/UI library (controllers, LiveViews, components, templates) and static assets under `apps/game_server_web/priv/static`.
- repository root: runnable host that starts the supervision tree and is the intended extension point for forks (routing, boot-time config, branding, root assets/static/theme/data).

### Running the app

- Dev entrypoint is the root host app: `mix dev.start` starts the repository-root `game_server_host` project.
- The Phoenix endpoint module is still `GameServerWeb.Endpoint` for compatibility (static paths, existing modules, UI library), but it is started by the host OTP application.

### Routing ownership / extension point

Routing is host-controlled without making `game_server_web` depend on `game_server_host` at compile time:

- Host router: `GameServerHost.Router` in `lib/game_server_host/router.ex`.
- Endpoint dispatch: `GameServerWeb.Endpoint` uses a small dispatch plug that reads `Application.get_env(:game_server_web, :router, GameServerWeb.Router)` and calls that module.
  - This avoids warnings like “`GameServerHost.Router.call/2 is undefined`” when compiling `game_server_web` in isolation.
- Host boot-time config: `GameServerHost.Application` sets `Application.put_env(:game_server_web, :router, GameServerHost.Router, persistent: true)` before starting the endpoint.

Fork guidance:

- Add/remove routes by editing `lib/game_server_host/router.ex`.
- If you *remove* upstream routes but the upstream UI still uses route helpers (`~p"..."`) pointing at `GameServerWeb.Router`, you can end up generating links for routes that the host no longer serves. If you want strict route removal, forks should adjust the UI accordingly (or provide replacement routes that match the UI’s expectations).### Phoenix v1.8 guidelines

- **Always** begin your LiveView templates with `<Layouts.app flash={@flash} ...>` which wraps all inner content
- The `MyAppWeb.Layouts` module is aliased in the `my_app_web.ex` file, so you can use it without needing to alias it again
- Anytime you run into errors with no `current_scope` assign:
  - You failed to follow the Authenticated Routes guidelines, or you failed to pass `current_scope` to `<Layouts.app>`
  - **Always** fix the `current_scope` error by moving your routes to the proper `live_session` and ensure you pass `current_scope` as needed
- Phoenix v1.8 moved the `<.flash_group>` component to the `Layouts` module. You are **forbidden** from calling `<.flash_group>` outside of the `layouts.ex` module
- Out of the box, `core_components.ex` imports an `<.icon name="hero-x-mark" class="w-5 h-5"/>` component for for hero icons. **Always** use the `<.icon>` component for icons, **never** use `Heroicons` modules or similar
- **Always** use the imported `<.input>` component for form inputs from `core_components.ex` when available. `<.input>` is imported and using it will will save steps and prevent errors
- If you override the default input classes (`<.input class="myclass px-2 py-1 rounded-lg">)`) class with your own values, no default classes are inherited, so your
custom classes must fully style the input

### JS and CSS guidelines

- **Use Tailwind CSS classes and custom CSS rules** to create polished, responsive, and visually stunning interfaces.
- Tailwindcss v4 **no longer needs a tailwind.config.js** and uses a new import syntax in `app.css`:

      @import "tailwindcss" source(none);
      @source "../css";
      @source "../js";
      # NOTE: in this repo (umbrella split), web UI code lives under apps/game_server_web
      @source "../../apps/game_server_web/lib";

- **Always use and maintain this import syntax** in the app.css file for projects generated with `phx.new`
- **Never** use `@apply` when writing raw css
- **Always** manually write your own tailwind-based components instead of using daisyUI for a unique, world-class design
- Out of the box **only the app.js and app.css bundles are supported**
  - You cannot reference an external vendor'd script `src` or link `href` in the layouts
  - You must import the vendor deps into app.js and app.css to use them
  - **Never write inline <script>custom js</script> tags within templates**

### UI/UX & design guidelines

- **Produce world-class UI designs** with a focus on usability, aesthetics, and modern design principles
- Implement **subtle micro-interactions** (e.g., button hover effects, and smooth transitions)
- Ensure **clean typography, spacing, and layout balance** for a refined, premium look
- Focus on **delightful details** like hover effects, loading states, and smooth page transitions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appsinacup/game_server](https://github.com/appsinacup/game_server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
