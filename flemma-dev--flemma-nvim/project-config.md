---
trigger: always_on
description: Flemma.nvim is a Neovim plugin for LLM-powered chat in `.chat` buffers. The buffer is the conversation — portable, re-parseable, and version-controllable. Keep each contribution focused, reversible, and well-documented so the next contributor can continue seamlessly.
---

# Repository Guidelines

Flemma.nvim is a Neovim plugin for LLM-powered chat in `.chat` buffers. The buffer is the conversation — portable, re-parseable, and version-controllable. Keep each contribution focused, reversible, and well-documented so the next contributor can continue seamlessly.

---

# Part 1: Architecture & Conventions

## Critical Rules

These are counter-default behaviors — violating them breaks the build or introduces bugs:

- **All JSON through `require("flemma.utilities.json")`** — never bare `vim.json.*` or `vim.fn.json_*` (JSON `null` becomes truthy `vim.NIL` instead of `nil`)
- **All structural buffer inspection through the AST** — never regex/substring matching on buffer lines
- **All `require()` calls at file top** — `make qa` enforces this
- **Full EmmyLua type annotations on all production code** — `make qa` enforces this
- **Never `---@diagnostic disable` as an easy fix** — use `---@cast`, `--[[@as type]]`, or restructure; suppressions only for genuine LuaLS limitations

## Design Principles

- **Flemma is stateless; the buffer is the state.** All conversation data, tool calls, and results must be fully represented in the buffer text. Never rely on in-memory state that would be lost when Neovim restarts or when a `.chat` file is shared. If you need to persist information (e.g., synthetic IDs for providers that don't supply them), embed it in the buffer format itself so it can be parsed back later. In-memory structures (`state.lua`) are ephemeral caches rebuilt from the buffer on demand — they are never the source of truth.

- **The outgoing request is a product of _(conversation, environment)_.** The buffer determines what was said; the environment determines how it's delivered. "Environment" means the config layer stack (SETUP/RUNTIME layers below FRONTMATTER), the tool registry, the personality builder's ambient state (`cwd`, `git_branch`, `date`, `time`, project context files on disk), template expression evaluation (`os.date`, `os.time`, `include()`, `math.random`), and model metadata from the registry. Given the same `.chat` buffer and the same environment, the request is deterministic. When adding new inputs to the request pipeline, decide which half they belong to — conversation state goes in the buffer, ambient context goes through the environment — and never mix the two.

- **All structural operations go through the AST.** The parsed AST (cached per buffer via `state.ast_cache`) is the only way to inspect conversation structure — roles, tool use/result blocks, thinking blocks, positions. If the AST lacks information you need, extend the AST rather than bypassing it. Direct buffer manipulation is only appropriate for content injection (tool results, streaming text) and UI concerns (spinners, extmarks).

- **Async/blocking work in the send pipeline goes through `flemma.readiness`.** Mirrors the `Confirmation` pattern in `lua/flemma/preprocessor/context.lua`: leaf code that would block on subprocess IO (e.g., `secrets.resolve`, `tools.get_for_prompt`) raises `error(readiness.Suspense.new(message, boundary))`. Orchestrators (`core.send_to_provider`, `usage.prefetch.fire_fetch`, `:Flemma usage:estimate`) wrap their pipeline in `pcall`, check `readiness.is_suspense(err)`, subscribe via `boundary:subscribe(cb)`, and retry the whole pipeline on completion. Boundaries are keyed by string (e.g., `secrets:vertex:access_token`) and shared — concurrent consumers of the same key share one in-flight runner. Never use `vim.system(cmd):wait()` in code reachable from the send pipeline — use the async `vim.system(cmd, opts, on_exit)` form behind a boundary.

- **EmmyLua type annotations are mandatory.** Every production file must have full LuaLS type coverage — `---@class`, `---@param`, `---@return`, `---@field`, `---@type` on all public and private functions, fields, and return values. New code without type annotations will not pass `make qa`.

- **Name modules according to their file path** (`lua/flemma/provider/adapters/openai.lua` → `flemma.provider.adapters.openai`). Public APIs belong in the module that owns the domain — tool registration lives in `flemma.tools`, provider registration in `flemma.provider.registry`, session access in `flemma.session`, etc. Don't pollute `init.lua` with single-use accessors; users require the specific module directly.

- **Verify claims before asserting them.** Commit messages, comments, and docs that make claims you haven't tested rot fast and mislead the next contributor. If you write "X is only reachable via Y," back it with a test.

- **Before mirroring a pattern, understand why it lives where it does.** Patterns are shaped by surrounding context — which capability branch owns them, which module, which provider. A mirror that ignores context becomes a leak into shared code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Flemma-Dev/flemma.nvim](https://github.com/Flemma-Dev/flemma.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
