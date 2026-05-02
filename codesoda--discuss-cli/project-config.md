---
trigger: always_on
description: Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.
---

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## Rust Patterns

- Shared fallible APIs should use `discuss::DiscussError` and `discuss::Result`, re-exported from `src/lib.rs`.
- Keep clap argument definitions in `src/cli.rs`; `src/main.rs` should stay thin and map app errors through `discuss::exit_code_for_error`.
- Config defaults and TOML parsing live in `src/config.rs`; use `Config::from_toml_str` when parsing file contents so errors preserve the config path and line/column location.
- Use `Config::resolve(ConfigOverrides)` for full layered config resolution. Internally, file/env layers are partial so omitted keys do not reset values from lower-priority layers.
- Runtime launch resolves `--port`/`--no-open`/`--history-dir`/`--no-save` through `ConfigOverrides` and binds exactly `127.0.0.1:<port-or-7777>` via `serve_with_ready`; do not add a free-port fallback because agent URLs must stay predictable.
- Browser launch/status helpers live in `src/launch.rs`; write the single `listening on http://127.0.0.1:<port>` line to stderr after bind, call `open::that` only when `auto_open` remains true, and log browser-open failures with tracing warnings instead of failing the session.
- `discuss <file>` emits `session.started` from `src/lib.rs` inside the `serve_with_ready` readiness callback before stderr announcement/browser open; payload keys are `url`, `source_file`, and `started_at`.
- Tracing initialization lives in `src/logging.rs`; `run` resolves `Config` and calls `init_tracing`, which must write only to the rolling log file because stdout is reserved for JSON events.
- Markdown rendering lives in `src/render.rs` as pure `render(&str) -> String`; configure Comrak there, and keep the dependency on `default-features = false` unless a future story explicitly needs CLI/syntax-highlighting features.
- Bundled page-shell rendering lives in `src/template.rs`; call `render_page(rendered_markdown, initial_state_json)` after markdown rendering to preserve `discuss.html` while injecting `#doc-content` and seeding `window.__DISCUSS_INITIAL_STATE__`.
- `src/template.rs` must target the real `#doc-content` section after `<body>` because the template's top instructional comment also mentions `<section id="doc-content">`.
- Bundled browser assets live in `assets/` and are exposed through `src/assets.rs`; `render_page` inlines the Mermaid shim, while `assets::mermaid_js()` provides the minified asset for later static routes.
- Static browser asset routes live in `src/server.rs`; keep them exact-path, serve from `src/assets.rs`, and include `Cache-Control: public, max-age=86400`.
- State protocol types live in `src/state/types.rs`; keep serde field names camelCase, serialize `ThreadId` transparently as a string, and encode new-thread draft anchor ranges as `"start-end"` JSON object keys.
- Process-local review state lives in `src/state/store.rs`; use `State::new_shared()` for `Arc<RwLock<State>>`, mutate through typed accessors, and call `snapshot()` for the active browser/API state while soft-deleted threads stay preserved internally.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codesoda/discuss-cli](https://github.com/codesoda/discuss-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
