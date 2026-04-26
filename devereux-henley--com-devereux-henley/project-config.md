---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Clojure Parenthesis Repair

The command `clj-paren-repair` is installed on your path.

Examples:
`clj-paren-repair <files>`
`clj-paren-repair path/to/file1.clj path/to/file2.clj path/to/file3.clj`

**IMPORTANT:** Do NOT try to manually repair parenthesis errors.
If you encounter unbalanced delimiters, run `clj-paren-repair` on the file
instead of attempting to fix them yourself. If the tool doesn't work,
report to the user that they need to fix the delimiter error manually.

The tool automatically formats files with cljfmt when it processes them.

## Commands

```bash
# Workspace health
clojure -M:poly info      # show components, bases, projects
clojure -M:poly check     # validate component boundaries
clojure -M:poly test      # run tests across affected units

# Build JARs
clojure -M:build -A api uber          # → target/rts-api.jar
clojure -M:build -A deploy-data uber  # → migrations CLI JAR

# Docker (from bases/rts-api/)
make build_docker
make run_docker
make ory_local            # create Ory Cloud tunnel for local auth

# Refresh game data seed files from RPFM (after a patch)
python3 scripts/update_from_rpfm.py --data-dir scripts/rpfm_data
```

See `docs/game-data.md` for the full RPFM data refresh workflow.

**REPL (primary dev workflow):** Jack-in from the repo root with `M-x cider-jack-in`. The `:dev` alias (configured in `.dir-locals.el`) puts all components and bases on the classpath.

**Claude's dev workspace:** `development/src/claude_workspace.clj` — Claude Code's own scratch namespace with system lifecycle helpers (`go!`, `halt!`, `restart!`) and migration helpers (`migrate!`, `rollback!`, `reset-db!`, `seed-db!`). Require it at the REPL with `(require 'claude-workspace)` or add evals directly to the scratch `comment` block. Keep dev helpers here rather than polluting `workspace.clj`.

```clojure
(go!)       ; start system — runs migrations + starts Jetty on :3001
(halt!)     ; stop
(restart!)  ; halt then go
```

**Running a specific test namespace:**

```bash
# From a base or component directory
clojure -A:test -e "(require 'clojure.test '<namespace>) (clojure.test/run-tests '<namespace>)"
```

Or load namespaces in the REPL and call `(clojure.test/run-all-tests)`.

## Architecture

This is a **Polylith monorepo** for an RTS tournament platform. Polylith enforces three kinds of artefacts:

- **components/** — shared units of behaviour, each with a public `interface` namespace
- **bases/** — runnable entry points that wire components together
- **projects/** — deployable combinations (`api`, `deploy-data`) with their own `deps.edn` and `build.clj`

### Component dependency order

```
rts-web → rts-domain → rts-data-access
                    ↘ rts-data (migrations)
rts-web, rts-domain, rts-data-access → http, jdbc, schema, content-negotiation
```

### Request lifecycle (rts-api base)

1. **Reitit** routes the request, coerces path/query/body via Malli schemas
2. **Muuntaja** negotiates content type (`application/json`, `application/hal+json`, `text/html`, `application/htmx+html`)
3. **Integrant**-managed handler (`init-key`) receives injected `{:db … :router …}`
4. Handler calls domain functions. Domain signals errors in two ways:
   - **Logic/validation errors** (budget exceeded, duplicate lord, etc.) — returned as a typed map `{:type :draft/add-error :message "…"}`; web handlers dispatch on `:type` to choose the HTTP status and body. **No try/catch in web handlers.**
   - **Infrastructure/missing-resource errors** — thrown as `ex-info` with `:error/kind` (`:error/missing`, `:error/invalid`, `:error/conflict`). These propagate through the stack and are caught by the Reitit exception middleware in `web.clj` (`exception-handlers`), which maps them to the appropriate HTTP status and renders an error page or JSON body.
5. On success, the **model transformer** walks the resource, resolves `:model/link` annotations into `_links` URLs using the reitit router
6. For HTML responses, a Selmer template is chosen by `:type` in the response body (dispatch map in `web.clj`)

### Key patterns

**HATEOAS everywhere.** Every resource carries `_links`. Foreign-key fields are annotated with `[:field {:model/link :route/name} :uuid]`; the transformer resolves the named route to a URL automatically.

**Schema separation.** *Entity schemas* (in `rts-data-access`) mirror DB columns. *Resource schemas* (in `rts-domain`) are what the API returns. Handlers map entity → resource (add `:type`, shape links, etc.).

**Integrant DI.** Each handler is an `integrant.core/init-key` method. System wiring lives in `configuration.clj`. Add a new handler by adding an `init-key` method, a `halt-key!` if needed, and a `ref` in the system map.

**SQL in resources.** Queries live in `.sql` files under `resources/<base>/sql/` and are loaded by `next.jdbc`. The `jdbc` component provides thin wrappers (`query-for-entity`, `insert!`, etc.) that apply camel-snake-kebab column mapping and the `sqlite-transformer`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Devereux-Henley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
