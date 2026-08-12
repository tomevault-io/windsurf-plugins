---
trigger: always_on
description: Rover is an **open-source Elixir library**, not an application: it gives Phoenix
---

Rover is an **open-source Elixir library**, not an application: it gives Phoenix
LiveView a `<.map>` component backed by OpenLayers. Everything below about
Phoenix and LiveView still applies to how the code is written, but remember that
the audience is other people's applications.

## Project guidelines

- Use `mix precommit` when you are done with all changes and fix any pending issues.
  It runs `format`, `compile --warnings-as-errors`, the ExUnit suite, the Node
  test suite, and rebuilds the JavaScript bundles.
- `mix dev` starts the playground on http://localhost:4020 (`dev/demo_live.ex`).
  `PORT=4021 mix dev` if that port is taken.
- `mix assets.test.browser` runs the Playwright suite against the playground. It is
  **not** in `precommit` (it needs a server and a browser) but it runs in CI, and it
  is the only place the canvas and the popup DOM are exercised at all. When you add
  a guard there, reintroduce the bug and watch it fail — a regression test nobody
  has seen go red proves nothing, and one of these did not actually guard what its
  name claimed until it was checked that way.
- `bin/ci` reproduces `.github/workflows/ci.yml` in a container against your working
  tree — useful to catch a red CI before pushing. `SKIP_BROWSER=1 bin/ci` skips the
  slow Playwright leg while iterating.
- Keep the dependency list minimal. Library consumers pay for every dep we add;
  anything only the playground needs goes under `only: :dev`.
- The JavaScript lives in `assets/js` and is bundled into `priv/static` by
  `assets/build.js`. **Those built files are committed** — they are what Hex
  ships, so an application with no npm can still use Rover. Rebuild them
  (`mix assets.build`) in any commit that touches `assets/`.
- The public vocabulary is `{latitude, longitude}`. The flip to OpenLayers'
  `[x, y]` happens only in `assets/js/coords.js`.
- **One documented exception:** `Rover.Shape` takes GeoJSON, which RFC 7946
  defines as `[longitude, latitude]`. Shape data is never typed by hand — it comes
  from `ST_AsGeoJSON`, a cadastral API, a routing service — so the standard wins
  there and only there. Latitude-first still governs markers, centres and every
  event payload.
- Shape geometry is diffed by a server-computed `:rev`, never by hashing
  coordinates on the client. A route is thousands of points; hashing it per update
  is the cost the reconciler exists to avoid.
- Markers and shapes are reconciled by `:id`. Any change that makes an update
  rebuild unrelated features is a regression — `assets/test/markers.test.js` and
  `assets/test/shapes.test.js` assert this by object identity.
- Popups are deliberately **not** `ol/Overlay`. An Overlay reparents its node into
  the map viewport, which lives inside `phx-update="ignore"`; LiveView would then
  be patching markup it no longer controls. `assets/js/popups.js` positions
  server-rendered nodes that never leave the outer element.

### Phoenix v1.8 guidelines

- **Always** begin your LiveView templates with `<Layouts.app flash={@flash} ...>` which wraps all inner content
- The `MyAppWeb.Layouts` module is aliased in the `my_app_web.ex` file, so you can use it without needing to alias it again
- Anytime you run into errors with no `current_scope` assign:
  - You failed to follow the Authenticated Routes guidelines, or you failed to pass `current_scope` to `<Layouts.app>`
  - **Always** fix the `current_scope` error by moving your routes to the proper `live_session` and ensure you pass `current_scope` as needed
- Phoenix v1.8 moved the `<.flash_group>` component to the `Layouts` module. You are **forbidden** from calling `<.flash_group>` outside of the `layouts.ex` module
- Out of the box, `core_components.ex` imports an `<.icon name="hero-x-mark" class="w-5 h-5"/>` component for hero icons. **Always** use the `<.icon>` component for icons, **never** use `Heroicons` modules or similar
- **Always** use the imported `<.input>` component for form inputs from `core_components.ex` when available. `<.input>` is imported and using it will save steps and prevent errors
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
- **Always** manually write your own tailwind-based components instead of using daisyUI for a unique, world-class design
- Out of the box **only the app.js and app.css bundles are supported**
  - You cannot reference an external vendor'd script `src` or link `href` in the layouts
  - You must import the vendor deps into app.js and app.css to use them
  - **Never write inline <script>custom js</script> tags within templates**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nseaSeb/rover](https://github.com/nseaSeb/rover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
