---
trigger: always_on
description: When generating or reviewing Pull Request titles and descriptions, you must adhere to the "Effective PRs" guidelines. Prioritize clarity, searchability, and narrative flow over snippets and bullet points.
---

# GitHub Copilot Instructions: Effective Pull Requests

When generating or reviewing Pull Request titles and descriptions, you must adhere to the "Effective PRs" guidelines. Prioritize clarity, searchability, and narrative flow over snippets and bullet points.

## 1. PR Titles
- **Autonomy:** Titles must "stand alone" and be descriptive enough for a reader to understand the scope without opening the PR or a Jira ticket.
- **Context:** Include the subcomponent or system affected at the start (e.g., `config: `).
- **Searchability:** Use keywords that a developer would use when searching for changes in this specific area later.
- **Format:** `Jira-ID [component]: [descriptive action]`.
- **Avoid:** Generic titles like "fix bug," "small fix," or "update deps."

## 2. PR Descriptions (The Narrative)
Do NOT use section headers (e.g., ## Why). Instead, separate distinct concerns into clear, cohesive paragraphs. Use Markdown backticks for all code symbols, class names, or file paths (e.g., `WidgetValidator`). Use prose for all explanations unless providing a literal list of items.

Follow this narrative structure:

- **Paragraph 1: What is changing?** Provide a high-level summary of what the PR accomplishes. Focus on the intent and the "first audience" (the reviewer). Mention if new features are introduced or if existing APIs/assumptions are being modified.
- **Paragraph 2: Why is this change needed?** Explain the "Big Why." Detail the motivating reason, such as fixing a breakage or fulfilling a requirement for a larger effort. Reference Jira issues or external resources to provide context on importance.
- **Paragraph 3: Why this particular approach?** Explain the "Little Why." Detail implementation choices and why they are superior to alternatives. Explain how the code fits into existing architectures (e.g., deriving from a specific base class).
- **Paragraph 4: What are the risks?** Explicitly state the risks or lack thereof. If risks exist (e.g., CPU spikes from regex), explain how they are mitigated (e.g., using a `SafeRE` library or timeouts). 
- **Paragraph 5: How has it been tested?** Detail the specific steps taken to ensure correctness. Mention new unit tests, regression tests, or fuzz tests. If risks were identified in the previous paragraph, explain how the tests specifically address those risks.
- **Paragraph 6: What's next?** (If applicable) Explain if this is part of a multi-stage rollout, such as a change currently disabled by a feature flag or a precursor to a follow-up PR.

# GitHub Copilot Instructions: Code Review Guidelines

When reviewing code in this repository, apply the following guidelines. These are distilled from recurring review feedback and represent the team's standards.

## 1. Error Handling (Snafu)
- Never use `.unwrap()` on fallible operations. Always propagate errors with `?` or return a descriptive error.
- Use Snafu idiomatically: prefer `.context(MySnafu { ... })` over `map_err`. Import `ResultExt` for the `.context()` method.
- Create domain-specific error types rather than reusing generic ones (e.g., don't use `RestError` for encryption failures -- create an encryption-specific error type).
- Error variant naming: use past-tense failure descriptions (e.g., `Base64DecodeFailed`, `IntegerParsingFailed`, not `Base64DecodeError`).
- Preserve error context: when converting errors, never discard the status code, original message, or causal chain.

## 2. Architecture and Design
- Make illegal states unrepresentable: use enums with associated data, typed wrappers, and non-optional fields instead of `Option` fields validated at runtime.
- Prefer state machines over boolean flags for tracking object lifecycle.
- Separation of concerns: don't couple storage/cloud layers with query layers. Keep FFI conversion at the FFI boundary, not deep inside business logic.
- Reuse existing patterns: follow `from_settings(settings: &dyn Settings)` for config structs. Wrap multi-field returns in named structs (like `SessionTokens`).
- Don't expand interface surface unnecessarily: prefer `connection_set_option` over adding new interface functions for every config knob.
- Extract helpers: when a cast or conversion pattern is repeated (e.g., handle-to-connection), introduce a helper function.

## 3. Concurrency and Safety
- File descriptor races: check file permissions, read/write, and ensure metadata on the same file descriptor. Never check-then-act with separate opens.
- Mutex guard lifetime: don't drop a mutex guard before completing the critical section. Premature drop causes race conditions (e.g., duplicate CRL downloads).
- Acquire locks once: if multiple fields are read from the same locked struct, acquire the mutex once rather than locking/unlocking repeatedly.
- Async vs blocking: never block the async executor thread. Use `tokio::task::block_in_place` for long-running blocking operations (e.g., keyring access). Use `tokio::time::sleep` instead of `std::thread::sleep` in async code.

## 4. Testing
- For cpp/odbc: flat tests, no test classes: avoid `class`-based test grouping. Keep tests as standalone functions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snowflakedb/universal-driver](https://github.com/snowflakedb/universal-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
