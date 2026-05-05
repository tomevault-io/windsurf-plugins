---
trigger: always_on
description: Validates the 253-built plugin works when loaded into newer IDEs at runtime.
---

# CLAUDE.md, AGENTS.md

Guidance for Claude Code when working with this repository. **Instructions here override default behavior.**

Never include AI as co-author or mention AI in commit messages.

## MUST DO

- Use IntelliJ MCP for everything where you can
- Never ignore warnings or errors — fix them properly
- No test-only branches (`isUnitTestMode`) — use correct IntelliJ actions (`writeIntentReadAction`, `writeCommandAction`)
- Tests must show reality — a failing test is better than a fake passing test. **Never remove, disable, or weaken a failing test**; fix the underlying issue instead
- No `@Suppress("DEPRECATION")` — find the non-deprecated replacement
- Prefer JSON libraries for JSON parsing/manipulation; only static final JSON constants may be hand-written as raw JSON strings
- **BANNED:** `runCatching{}.onFailure{}` — use `try { } catch (e: Exception) { }` instead. Other `runCatching` uses (`.getOrNull()`, `.getOrDefault()`) are fine
- **Fail fast and log problems.** Never swallow exceptions with empty `catch` blocks or `catch (_: Exception) {}`. Every catch must either rethrow, log the error with `System.err.println` / `logger.error`, or both. Silent failure hides the root cause and wastes iteration cycles. When something can fail (DLL loading, file I/O, network calls), log what was attempted and what went wrong
- **BANNED:** Production code and tests must never reference or depend on `run-agent.sh` or `docs/run-agent.sh`. These scripts are tools for humans and AI agents to use manually during development (peer reviews, research, etc.), not for programmatic execution from project code. Code should implement agent integrations directly using CLI flags and arguments. `run-agent.sh` must **never** be installed inside Docker containers (no `COPY run-agent.sh` or `RUN chmod +x ... run-agent.sh` in Dockerfiles)
- **BANNED:** Gradle build files must never reach into another subproject's `build/` directory directly. Use Gradle dependency configurations to share artifacts between subprojects. Fail fast with a clear `require()`/`error()` — no silent fallbacks that hide misconfiguration
- **BANNED:** Do NOT use `append("\n")` or `append("...\n")` tricks to work around the `NoLargeInlineStringsTest` lint rule. When a `buildString { }` block exceeds the consecutive-`appendLine` limit, the correct fix is to move the content to `src/main/prompts/` resource files and reference them via article URIs — not to sprinkle `append("\n")` calls to artificially break the line count.
- **BANNED:** Do NOT hardcode `mcp-steroid://...` resource URIs as string literals in production Kotlin code. Use the generated prompt article class instead: `XxxPromptArticle().uri` (from `com.jonnyzzz.mcpSteroid.prompts.generated.*`). This ensures URIs stay in sync with prompt files and break at compile time if renamed. Enforced by `NoHardcodedMcpSteroidUriUsageTest`. See `FetchResourceToolHandler.kt` for the correct pattern.
- Log new ideas/tasks in TODO* files (TODO.md, TODO-*.md)
- **No infrastructure workarounds**: when tests fail due to infrastructure limitations (missing Docker socket, missing CLI, wrong JDK, missing native libraries), fix the infrastructure — mount Docker socket, install Docker CLI, configure JDK, install the dependency. Do NOT add code that detects the limitation and silently skips tests or changes behavior. A failing test that reveals a real problem is better than a passing test that hides it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonnyzzz/mcp-steroid](https://github.com/jonnyzzz/mcp-steroid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
