---
trigger: always_on
description: - **Use the MCP SDK's server builder (FastMCP)** – Initialize a `FastMCP` server instance (e.g. in a main `server.py`) with a clear name for your application. Avoid writing custom networking or JSON handling; let the SDK handle connection management and message routing.
---

## Architecture & Code Structure
- **Use the MCP SDK's server builder (FastMCP)** – Initialize a `FastMCP` server instance (e.g. in a main `server.py`) with a clear name for your application. Avoid writing custom networking or JSON handling; let the SDK handle connection management and message routing.
- **Separation of concerns** – Organize code into modules for distinct purposes. Keep MCP interface code (server setup, tool/resource definitions) separate from business logic (strategy algorithms, data loading, backtest engine). For example, have a module for strategy logic and another for MCP server initialization that calls into that logic.
- **Leverage SDK conventions** – Define MCP **tools** and **resources** using the provided decorators (e.g. `@mcp.tool()`, `@mcp.resource()`) instead of manual routing. This ensures the server advertises capabilities correctly to the client and maintains protocol compliance. Use the SDK’s structure (prompts, tools, resources as needed) to align with MCP’s expected architecture.
- **Lifecycle management** – If your server needs setup/teardown (database connections, data pre-loading), use the `lifespan` context provided by FastMCP (or similar startup hooks) rather than running setup code at import time. This yields a cleaner architecture and integrates with MCP’s lifecycle (init and shutdown events) gracefully.
- **Minimal server entry-point** – Keep the code that launches the server lightweight. For example, use an `if __name__ == "__main__":` block or a small CLI to start the MCP server (possibly using `mcp.run()` or `uvicorn` if using HTTP). This makes it easier to start the server in different modes (dev, production) and to test components in isolation.

## Agent Registration & Session Management
- **Explicit agent registration** – Provide tools or functions for registering a new trading strategy/agent into the system (e.g. a `register_strategy(name, config)` tool). When an agent is registered, store its definition (parameters, possibly code or identifier) in an internal registry. Confirm registration by returning a unique ID or name to reference that strategy later.
- **Session isolation** – Design the server to handle multiple sessions or users without interference. Utilize MCP’s session features to keep each session’s data separate. For example, maintain a session-specific structure (dictionary or context object) that holds the strategies and state for that session. **Do not use global state** for per-session data; instead, use the `Context` or session storage provided by the SDK (e.g. `ctx.session` or similar) to tie data to a session.
- **Track session state** – Maintain clear state for each strategy/agent within a session: whether it’s idle, running a backtest, or completed. If a strategy is running, subsequent actions (like checking results) should reference the correct session and strategy. Use unique keys (like session_id + strategy_id) if storing in a global registry to avoid collisions, but prefer built-in session context when available.
- **Cleanup and lifecycle** – Handle the removal or resetting of strategies when appropriate. For example, if a session ends or an agent is unregistered, ensure its state is cleaned up from the registry. This prevents stale data from accumulating and ensures each new session starts fresh or with only intended persisted state.
- **Concurrent execution considerations** – If multiple agents or multiple sessions might run backtests concurrently, ensure that shared resources are managed safely. Use locks or asyncio synchronization for shared data, or better, avoid sharing mutable global data altogether by scoping it per session/agent. This will prevent race conditions when traders run strategies in parallel.

## Strategy Execution
- **Encapsulate backtest logic** – Implement the core backtest runner in a dedicated component (function or class) separate from the MCP interface. The tool that starts a backtest should call this component, passing in the strategy details and data. This separation makes the logic reusable and easier to test (you can unit test the backtest runner without MCP).
- **Asynchronous or background execution** – If a backtest is time-consuming, run it asynchronously to avoid blocking the MCP server event loop. Mark the tool function as `async` and use `await` for the backtest function, or offload heavy computation to a background thread or task. This allows the server to handle other messages (or heartbeats) and even stream progress without freezing.
- **Progress updates** – For long-running strategies, consider providing feedback during execution. For example, periodically log progress or send interim results (if supported by MCP, e.g. via an observable resource or notifications). This keeps the user informed and helps avoid timeouts. Design the backtest runner to yield or callback progress (every N ticks or at major steps) so the AI agent can relay this to the user if needed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tienan92it/binance-mcp](https://github.com/tienan92it/binance-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
