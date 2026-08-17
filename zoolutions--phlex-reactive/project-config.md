---
trigger: always_on
description: Reactive [Phlex](https://www.phlex.fun) components for Rails — Livewire-style
---

# phlex-reactive

Reactive [Phlex](https://www.phlex.fun) components for Rails — Livewire-style
actions and live cross-tab updates, without writing Stimulus controllers or
hand-picking Turbo Stream targets.

## Tech Stack

- **Ruby**: >= 3.4 | **Rails**: >= 7.1
- **Rendering**: phlex-rails (Phlex 2)
- **Transport**: turbo-rails (Turbo Streams); [pgbus](https://github.com/mhenrixon/pgbus) optional for Postgres SSE
- **Client**: one generic Stimulus controller (no per-feature JS)
- **Autoloading**: zeitwerk
- **Testing**: RSpec + Capybara/Playwright (via `spec/dummy`)
- **Linting**: RuboCop (`rubocop`) — all new cops on; teaches modern Ruby (e.g. `it` block param)

## Critical Rules

### Never Do
1. **NO hand-picked Turbo Stream targets** — a component self-targets via its stable `#id`
2. **NO shipping component STATE to the client** — the DOM carries a *signed identity* (`{c, gid}` or `{c, state}`), never raw state
3. **NO trusting client input for authorization** — the signature proves the token is ours, NOT that this user may act; authorize inside the action
4. **NO undeclared actions** — only methods declared with `action :name` are invokable (default-deny)
5. **NO raw mass assignment** — action params pass through the declared schema (`params: { x: :integer }`)
6. **NO hard dependency on pgbus** — broadcasts route through `Turbo::StreamsChannel`; phlex-reactive must work on Action Cable OR pgbus
7. **NO `dom_id` (Phlex render-time helper) inside `#id`** — `#id` runs before render; use `Streamable#dom_id` (delegates to `ActionView::RecordIdentifier`)
8. **NO bare `**on(...)` merged with another `data:`/`class:`** — use Phlex `mix(on(:x), data: {...})` or the extra hash clobbers `on`'s `data:`

### Always Do
1. **TDD**: Write tests BEFORE implementation (RED → GREEN → REFACTOR)
2. **Authorize every mutating action** — `authorize! @record, :update?` (register the error in `Phlex::Reactive.authorization_errors`)
3. **Declare a param schema** for any action that takes input
4. **Re-render through a real view context** — go through `Phlex::Reactive.renderer` / the controller, never a fabricated context (dom_id/url_for/t()/csrf must work)
5. **Capability-detect pgbus features at runtime** — probe the actual keyword (`broadcast.parameters` includes `:exclude`), never `defined?(Pgbus)` alone or a version string
6. **Degrade gracefully** — every pgbus-only feature must no-op or fall back when pgbus is absent
7. **Control the reply via the return value** — return a `Phlex::Reactive::Response` (`replace`/`update`/`remove`/`redirect`/`with`, chain `.flash(level, content)` / `.stream(...)`) to govern the actor's HTTP reply; returning anything else keeps the implicit single replace. See the README "Controlling the action's reply" section and `lib/phlex/reactive/response.rb`.
8. **Measure performance, don't guess** — any hot-path change (render, token signing, param coercion, broadcast, client dispatch) ships with a same-machine before/after from `rake bench` (or `/perf`). No speedup claim without a measured baseline. Report throughput AND allocations; distinguish a method-level win from a request-level one. See `.claude/rules/performance.md` and the performance page (`docs/app/views/docs/pages/performance.rb`).

## Commands

```bash
bundle exec rspec spec/phlex spec/requests   # Fast suite (unit + request + broadcast)
bundle exec rspec spec/system                # Browser suite (Playwright; Puma default, CAPYBARA_SERVER=falcon for the async server)
bundle exec rake spec:system_servers         # Browser suite under BOTH real servers (puma + falcon)
bundle exec rubocop                          # Lint (rubocop -A to autocorrect)
bundle exec rake                             # spec + rubocop
bundle exec rake bench                        # Performance micro-benches (render, token, coerce_params)
bundle exec rake bench:request                # End-to-end request-cycle bench (derailed)
rake build:js                                 # Rebuild the minified client (.min.js + .map) after editing reactive_controller.js
rake build:js_check                           # CI drift guard: committed .min.js must match a fresh build
```

### Editing the client runtime (`reactive_controller.js` / `confirm.js` / `compute.js` / `inspect.js`)

The gem ships the **minified** build, and the browser suite runs that same
minified build (the dummy vendors it). So a source edit is a THREE-file change:

```bash
rake build:js                                 # regenerate the .min.js + .map from source
cp app/javascript/phlex/reactive/reactive_controller.min.js \
   spec/dummy/public/vendor/reactive_controller.js   # re-sync the vendored copy (same for confirm/compute/inspect)
bun test spec/javascript                      # JS unit suite
```

Commit the source, the rebuilt `.min.js`/`.map`, AND the re-synced vendored copy
together. Two guards enforce it: `rake build:js_check` (committed min build matches
a fresh build) and `spec/phlex/vendored_controller_sync_spec.rb` (vendored copy is
byte-identical to the shipped `.min.js`). Its failure message prints the exact
re-sync command.

## Slash Commands

| Command | Purpose |
|---------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zoolutions/phlex-reactive](https://github.com/zoolutions/phlex-reactive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
