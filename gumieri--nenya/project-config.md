---
trigger: always_on
description: Nenya is a lightweight, highly secure AI API Gateway/Proxy written in Go. It acts as a transparent middleware between local AI coding clients (like OpenCode/Aider) and upstream LLM providers (Anthropic, Google Gemini, DeepSeek, Mistral, xAI, OpenRouter, and many more).
---

# Nenya - AI Agent Instructions

## Project Overview
Nenya is a lightweight, highly secure AI API Gateway/Proxy written in Go. It acts as a transparent middleware between local AI coding clients (like OpenCode/Aider) and upstream LLM providers (Anthropic, Google Gemini, DeepSeek, Mistral, xAI, OpenRouter, and many more). 

Its primary superpower is the **"Bouncer" mechanism**: intercepting massive HTTP payloads, routing them to a local Ollama instance (`qwen2.5-coder`) for summarization and PII/credential redaction, and forwarding the sanitized, much smaller payload to the upstream cloud AI using Server-Sent Events (SSE) streaming.

## Agent Role & Persona
You are acting as a **Senior Go Security Engineer and Network Architect**. Your code must be production-ready, highly performant, and paranoid about security and memory leaks.

## Strict Engineering Guidelines

### 1. Language & Communication
- **English Only:** All code, variables, functions, comments, commit messages, and documentation MUST be written in English.
- **No Yapping:** When generating code, output only the requested changes or files. Keep explanations brief and technical.

### 2. Go Architecture & OOP Patterns
- Follow Object-Oriented patterns via Go structs and receiver methods.
- **No Global Variables:** Encapsulate state inside structs (e.g., `NenyaGateway` holding the `Config` and `http.Client`).
- Use Dependency Injection where appropriate.
- Keep the `main.go` clean; delegate business logic to receiver methods.

### 3. Dependency Policy & Tech Stack
- The project relies exclusively on the Go Standard Library (`net/http`, `encoding/json`, `io`, `bytes`, `regexp`, `sort`).
- **Zero external dependencies.** DO NOT import any third-party packages without explicit human authorization.

### 4. Hardcore Security Rules (CRITICAL)
- **Timeouts:** NEVER use the default `http.Client` or `http.ListenAndServe`. Always explicitly define `ReadTimeout`, `WriteTimeout`, `IdleTimeout`, and Client `Timeout` to prevent resource exhaustion and hanging connections.
- **Body Limits:** Always wrap incoming requests with `http.MaxBytesReader` to prevent memory exhaustion attacks (DoS) from massive payloads.
- **Header Sanitization:** When proxying requests, strip hop-by-hop headers (like `Connection`, `Content-Length`) to prevent HTTP desync attacks. Pass only necessary headers (e.g., `Authorization`).
- **Error Handling:** Never expose internal stack traces to the HTTP response. Log errors internally and return standard HTTP status codes.

### 5. Context Package Standards (CRITICAL)
- **Request Context:** Always use `r.Context()` from `http.Request` as the root context for request-scoped operations. Thread it through the entire call stack.
- **Timeout Enforcement:** Apply appropriate timeouts to all outbound calls:
  - Upstream requests: Use `provider.TimeoutSeconds` from config
  - MCP operations: Use configured timeouts (30s for tool calls, 10s for auto-search)
  - Health checks: Use short timeouts (5s)
  - Long-running loops: Use overall deadlines (5min for MCP multi-turn loop)
- **Context Propagation:** Functions doing I/O or long-running work MUST accept `context.Context` as their first parameter. Never create a new `context.Background()` mid-request unless the operation is fire-and-forget and must outlive the request.
- **Goroutine Lifecycle:** All goroutines must respect cancellation:
  - Request-scoped goroutines: Use the request context or a derived context
  - Background workers: Use dedicated cancellation channels (`closeCh`, `stopCh`)
  - Fire-and-forget operations: Use `context.Background()` with explicit timeout
- **Loop Cancellation:** Long-running loops MUST check for cancellation:
  - Use `select { case <-ctx.Done(): }` for blocking operations
  - Check `ctx.Err()` in loop bodies for non-blocking operations
  - Never have infinite loops without cancellation checks
- **HTTP Requests:** Always use `http.NewRequestWithContext(ctx, ...)` — never `http.NewRequest` (deprecated)
- **I/O Operations:** Use context-aware I/O functions (e.g., `copyStream(ctx, dst, src, buf)`) instead of bare `io.Copy`
- **Anti-Patterns to Avoid:**
  - `context.TODO()` — never use, always have a clear parent
  - `context.WithValue()` — avoid for request-scoped data; use struct fields or parameters
  - `context.Background()` mid-request — only use for top-level contexts or fire-and-forget goroutines
  - Ignoring `ctx.Done()` — always respect cancellation signals

### 6. Core Workflows to Maintain

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gumieri/nenya](https://github.com/gumieri/nenya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
