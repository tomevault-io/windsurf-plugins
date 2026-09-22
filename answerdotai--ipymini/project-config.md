---
trigger: always_on
description: ipymini supplies Python language semantics to the Rust [kernmini](https://github.com/AnswerDotAI/kernmini) engine. IPython handles execution, history, completion, inspection, display, magics, configuration, and debugging. Kernmini handles Jupyter transport, messages, queues, output, stdin, interrupts, subshells, and lifecycle.
---

# Developer guide

ipymini supplies Python language semantics to the Rust [kernmini](https://github.com/AnswerDotAI/kernmini) engine. IPython handles execution, history, completion, inspection, display, magics, configuration, and debugging. Kernmini handles Jupyter transport, messages, queues, output, stdin, interrupts, subshells, and lifecycle.

The user-facing installation and kernelspec instructions are in `README.md`. Kernmini's `DEV.md` documents the language boundary and protocol engine.

## Project goals

- Keep the Jupyter engine in Rust and Python semantics in IPython.
- Use IPython machinery instead of reproducing Python behavior.
- Match ipykernel where clients can tell: message content, output ordering, history, inspection, completion, comms, debugging, and interrupts.
- Avoid traitlets and Tornado in the kernel host. IPython configuration remains supported through `InteractiveShellApp`.

## Source layout

- `ipymini/kernel.py`: a shell factory passed to the synchronous Rust-backed `kernmini.run_kernel`.
- `ipymini/shell/`: the IPython language adapter: execution, completion, inspection, history, comm binding, and debugging.
- `ipymini/term/`: Python stdout/stderr, input, display, and `get_ipython()` capture.
- `ipymini/debug/`: DAP/debugpy integration and ipykernel-compatible cell filenames.
- `ipymini/__main__.py`: CLI entry and kernelspec installation.
- `tests/`: protocol and behavioral integration tests, including unmodified jupyter_client clients.

## Startup and sessions

`ipymini.kernel.run_kernel` creates a closure around one shared namespace and passes it to kernmini. The first factory call creates the parent `InteractiveShell` singleton. Child calls create independent `InteractiveShell` instances sharing that namespace and the parent's in-memory history.

The parent shell runs on a persistent loopmini loop in the Python main thread. Each child runs on its own OS thread and persistent loopmini loop. Kernmini's Tokio runtime drives protocol and control work independently, so a synchronous Python cell cannot block the kernel engine or another subshell.

The executable requests process-group ownership. On POSIX this isolates the kernel and lets kernmini terminate user-created child processes during shutdown. Kernmini also watches the original parent PID. Embedded callers can disable process-group ownership, but the ipymini executable does not.

## Life of an execute request

The Rust engine validates and queues the request, publishes `busy` and `execute_input`, then calls `MiniShell.execute` with an execution-scoped context. The PyO3 adapter installs live stream, display, and input senders on each shell and stores the current Rust `ExecutionContext` in a Python ContextVar.

`MiniShell.execute` runs the cell through IPython's `run_cell_async`. Its execution context resets capture state and binds this shell's IPython instance, stdout, stderr, input, and display hooks. The display hook publishes each expression result through kernmini as it occurs, including multiple results with `ast_node_interactivity='all'`. The returned snapshot supplies errors, user expressions, and payload; kernmini publishes the reply before `idle`. Standalone shells without a bound kernel retain the last result in their snapshot.

Synchronous Python records its thread ID while running. Async cells remain ordinary loopmini tasks. Kernmini uses those two facts to inject `KeyboardInterrupt` into synchronous Python or cancel an async task without allowing SIGINT to escape the host loop.

## Output, input, and context

`term/io.py` installs process-wide dispatchers for `sys.stdout`, `sys.stderr`, `input`, `getpass`, and `get_ipython`. Their targets come from execution ContextVars. `threading.Thread.start` and `ThreadPoolExecutor.submit` copy the current context, so output from user-created threads remains attributed to the cell that created them.

`MiniStream` is a file-like stdout/stderr sink. During kernel execution it sends complete lines through kernmini's live stream callback; bare-shell unit tests can instead retain and coalesce events. `MiniDisplayPublisher` and `MiniDisplayHook` publish rich displays and expression values. The display hook flushes pending stream text before publishing a result.

The adapter's input callback crosses into Rust, which sends `input_request` to the correct client and blocks only the calling Python thread until `input_reply`. An interrupt completes the request with `KeyboardInterrupt`.

## Concurrent execution

Each language session executes one cell at a time. Completion, inspection, history, comms, debugging, and control requests remain responsive while it runs.

`subshell()` temporarily routes subsequent execute requests from the same client session to a child shell. The child has its own thread and IPython instance but shares the namespace. This allows genuinely concurrent synchronous work without requiring frontend JEP 91 support.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnswerDotAI/ipymini](https://github.com/AnswerDotAI/ipymini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
