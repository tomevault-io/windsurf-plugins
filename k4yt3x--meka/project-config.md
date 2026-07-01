---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Rust Coding Guidelines

- Prioritize code correctness and clarity. Speed and efficiency are secondary priorities unless otherwise specified.
- Do not write organizational or comments that summarize the code. Comments should only be written in order to explain "why" the code is written in some way in the case there is a reason that is tricky / non-obvious.
- Never hand-wrap comments. Write each comment (and doc comment) as one line per paragraph and let `cargo +nightly fmt` wrap it (`.rustfmt.toml` has `wrap_comments = true`). If fmt's wrap lands awkwardly, reword the prose rather than inserting a manual break.
- Prefer implementing functionality in existing files unless it is a new logical component. Avoid creating many small files.
- Avoid using functions that panic like `unwrap()`, instead use mechanisms like `?` to propagate errors.
- Be careful with operations like indexing which may panic if the indexes are out of bounds.
- Never silently discard errors with `let _ =` on fallible operations. Always handle errors appropriately:
    - Propagate errors with `?` when the calling function should handle them
    - Use `.log_err()` or similar when you need to ignore errors but want visibility
    - Use explicit error handling with `match` or `if let Err(...)` when you need custom logic
    - Example: avoid `let _ = client.request(...).await?;` - use `client.request(...).await?;` instead
- When implementing async operations that may fail, ensure errors propagate to the UI layer so users get meaningful feedback.
- Never create files with `mod.rs` paths - prefer `src/some_module.rs` instead of `src/some_module/mod.rs`.
- When creating new crates, prefer specifying the library root path in `Cargo.toml` using `[lib] path = "...rs"` instead of the default `lib.rs`, to maintain consistent and descriptive naming (e.g., `gpui.rs` or `main.rs`).
- Avoid creative additions unless explicitly requested
- Use full words for variable names (no abbreviations like "q" for "queue")
- Use variable shadowing to scope clones in async contexts for clarity, minimizing the lifetime of borrowed references.
  Example:
    ```rust
    executor.spawn({
        let task_ran = task_ran.clone();
        async move {
            *task_ran.borrow_mut() = true;
        }
    });
    ```

## Logging and output

`meka` maintains a strict split between *CLI output* and *tracing logs*. The test is simple: **if the user doesn't have to see this to use the command, it belongs in `tracing`**. Default log level is `warn`, so `info!` / `debug!` are silent unless the user passes `-v`, `-vv`, or `RUST_LOG`. Aim for "quiet on success" (the Unix convention).

**Use `println!` / `eprintln!` only when the output is unavoidable:**

- **Requested data**: what the user literally ran the command to get: the `meka mcp list` table, `meka mcp get` details, `meka session list` session rows, `meka session export` markdown on stdout, `print_help`.
- **Actionable content the user must copy/type/visit**: OAuth authorisation URLs, callback paste prompts, elicitation form fields, setup-wizard prompts.
- **REPL command output**: `/permission`, `/session`, `/cd` errors, `!cmd` status, tool-use indicators, streaming assistant markdown, thinking blocks, `Unknown command` feedback.
- **Hard errors** propagated back to the user with context (`render::render_error`, clap-side validation errors).
- Use `stdout` (`println!`) for parseable command output a script might consume; `stderr` (`eprintln!`) for prompts, live UI, and contract errors.

### `stdout` vs `stderr`

When `println!` / `eprintln!` *is* the right call (the output is unavoidable per the list above), the choice of stream is not a style decision; it's a contract:

- **`stdout` (`println!`, `print!`)**: only the data the user invoked the command to obtain. Examples: the agent's streamed assistant response, an `meka session list` table, an `meka session export -` markdown body, an `meka skill show` body, `meka mcp list` / `mcp get` / `mcp tools` rows.
- **`stderr` (`eprintln!`, `eprint!`)**: everything else: tool-call indicators, thinking blocks, todo lists, spacing newlines, status confirmations, hints, errors, interrupt notices, setup-wizard prompts, OAuth URLs, REPL UI feedback (`/permission`, `/cd`, `Unknown command`, approval prompts, `!cmd` exit-code messages).

**Litmus test:** `meka ... 2>/dev/null | next-tool` should leave only the requested data on stdout. If a user can't usefully pipe the output, your `println!()` is probably an `eprintln!()`.

The streaming markdown renderer (`render::StreamingRenderer`) writes to stdout because the assistant response *is* the requested output for an agent turn. Every other helper in `render.rs` (`render_session_id`, `render_hint`, `render_error`, `render_thinking_block`, `render_todo_list`, `render_tool_indicator`) and every spacing-blank-line emitted around them goes to stderr.

**Use `tracing` for everything else:**

- `error!`: unrecoverable failure about to propagate up as an `MekaError`. Rare; the `?` operator usually already carries the info.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k4yt3x/meka](https://github.com/k4yt3x/meka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
