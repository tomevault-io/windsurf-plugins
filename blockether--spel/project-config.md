---
trigger: always_on
description: - ALWAYS `load_skills=["spel"]` for browser tasks. `skill(name="spel")` first
---

# Agent Instructions

## Critical Rules

Browser Automation:
- ALWAYS `load_skills=["spel"]` for browser tasks. `skill(name="spel")` first
- NEVER `load_skills=["playwright"]` or `load_skills=["dev-browser"]` — disabled

Snapshots:
- ALWAYS `snapshot -i` when browsing — clickable elements only
- Full `snapshot` only when need text/structure
- `snapshot -i -c` → compact interactive (drops bare role lines)
- Read snapshot BEFORE clicking — no blind eval-js
- Click by ref: `spel click @eXXXX` — no CSS selectors or eval-js
- Skip REKLAMA in search results — ads, unrelated

SCI Bindings:
- NEVER inline `(fn ...)` lambdas for SCI fns — ALWAYS `defn` + docstring
- Every SCI fn = named `defn` (e.g. `sci-thread-sleep`, `sci-viewport-size`)
- Lambdas → break `gen-docs` → hidden from `FULL_API.md` → undebuggable
- Applies: `:bindings` map values, `make-ns-map` entries, `core/` stubs

Daemon Session Isolation:
- ALWAYS named session in dev/test — NEVER default "default"
- Generate: `SESSION=agent-$(date +%s)`
- Use on every cmd: `spel --session $SESSION open <url>`
- Teardown: `spel --session $SESSION close` — kills ONLY your session
- NEVER `spel close` without `--session` — kills user's default
- Verify: `spel session list`

Paren Repair:
- NEVER fix unbalanced parens by hand — `clj-paren-repair path/to/file.clj`

Linting:
- ALWAYS `lsp_diagnostics` on changed files — `clj-kondo` CLI NOT installed
- Library — unused public vars = intentional API surface, do NOT remove

Templates:
- NEVER edit `.opencode/agents/`, `.opencode/skills/`, `.opencode/prompts/`
- ALWAYS edit `resources/com/blockether/spel/templates/` → regenerate

Versioning:
- Single truth: `resources/SPEL_VERSION` (bare semver, no `v` prefix)
- NEVER hardcode — `(slurp (io/resource "SPEL_VERSION"))` + `str/trim`

API Policy:
- Pre-1.0: break callers freely, no shims, no deprecation

Feature Dev Order (Library → SCI → CLI):
- ALWAYS **library first** (`page.clj`, `input.clj`, `locator.clj`) — pure Clojure fns wrapping Playwright Java
- Then **SCI** (`sci_env.clj`) — session-atom convenience (implicit `@!page`, `@!context`) → interactive API
- Then **CLI/Daemon** (`daemon.clj` dispatch + `cli.clj` args) — JSON over Unix socket
- Library = **truth** — SCI + CLI REUSE library fns, never reimplement
- SCI → interactivity; CLI → discoverability (help, flags, JSON)

Screenshots:
- ALWAYS show screenshots for visual/UI changes
- HTML/CSS/template change → screenshot → display
- No visual change done without proof

## Testing

Every change = tests. `defdescribe`/`describe`/`it`/`expect` from `com.blockether.spel.allure`.

| Source | Test |
|---|---|
| `sci_env.clj` | `cli_integration_test.clj` → `sci-eval-integration-test` |
| `daemon.clj` | `cli_integration_test.clj` |
| `cli.clj` | `cli_test.clj` |
| `native.clj` (new CLI cmd) | `test-cli.sh` — `assert_jq` / `assert_contains` |
| `native.clj` (tool cmd) | `test-cli.sh` — minimum `--help` |
| Other | `*_test.clj` (e.g. `page.clj` → `page_test.clj`) |

### clojure.test (alt to Lazytest)

`clojure.test` + Allure via `allure-reporter`. Tests in `test/com/blockether/spel/ct/`.

```clojure
(ns my-app.test
  (:require [clojure.test :refer [deftest testing is]]
            [com.blockether.spel.allure :as allure]
            [com.blockether.spel.core :as core]))

(deftest my-test
  (allure/epic "My Epic")
  (testing "something"
    (core/with-testing-page [pg]
      (is (= 1 1)))))
```

Run: `clojure -M:test-ct`

### HTML test pages (`test_server.clj`)

Local HTTP test server at `test/com/blockether/spel/test_server.clj`.
New page:

1. `def ^:private` HTML string in `test_server.clj`
2. Route in `make-handler` cond: `(and (= "GET" method) (= "/my-page" path))`
3. Navigate: `(nav! "/my-page")` (Lazytest) or `(page/navigate pg (str ts/*test-server-url* "/my-page"))` (clojure.test)

| Route | Purpose |
|---|---|
| `/test-page` | Form elements |
| `/keyboard-page` | Keyboard events (`keydown` → `#last-key`, `#key-log`) |
| `/dialog-page` | Alert/confirm/prompt dialogs |
| `/second-page` | Nav target (back link) |
| `/iframe-page` | IFrame embedding test-page |
| `/redirect-page` | 301 → test-page |
| `/echo` | Echo req as JSON |
| `/health` | GET → `{"status":"ok"}`, HEAD → 200 |
| `/status/N` | HTTP status N |
| `/slow` | 2s delay |
| `/scrollable-page` | Scrollable containers (overflow:auto/scroll/hidden/visible) |

**Verify behavior, not "no error"** — HTML pages with observable DOM state. Never test return values only.

### CLI bash (`test-cli.sh`)
New CLI cmd/daemon action → new section:
- `section "Name (N)"` before SUMMARY
- `assert_jq`/`assert_jq_eq`/`assert_jq_contains` for `--json`
- `assert_contains` for text (e.g. `--help`)
- `TEMP_FILES+=("path")` for cleanup
- `"$SPEL"` not `$SPEL`
- Tool cmds (stitch, codegen, init-agents, ci-assemble, merge-reports, show-trace) MUST have `--help`

## Commands

```bash
make test                    # ALL: lazytest + CLI bash
make test-cli                # CLI bash only
make test-cli-clj            # CLI Clojure integration only
make format                  # auto-format
make lint                    # clojure-lsp --raw
make validate-safe-graal     # reflection/boxed-math check
make gen-docs                # regen FULL_API.md (before install-local)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blockether/spel](https://github.com/Blockether/spel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
