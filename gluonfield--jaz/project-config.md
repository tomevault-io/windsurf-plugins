---
trigger: always_on
description: - Keep code and JSON minimal. Each line of code should fight for its existence; every field and line must earn its place.
---

# Engineering Rules

- Use Go 1.26.
- Keep code and JSON minimal. Each line of code should fight for its existence; every field and line must earn its place.
- Prefer implementations that reduce total code over ones that add more. Adding lines is a cost to justify; a good fix often deletes code, collapses branches, or moves an invariant to the layer that already owns it.
- Trim strings at real input boundaries only: user input, config files, env vars, HTTP payloads, CLI args, and persisted loose text. Do not sprinkle `strings.TrimSpace` over internal constants, typed IDs, enum values, or values that have already crossed a validation boundary.
- When there's an opportunity for dramatic simplification or restructuring, bring it up. Favor "code judo" moves that delete layers, unify shapes, collapse special cases, or make the design inevitable over incremental patches.
- Bug fixes should first look for deletion or correction of the underlying contract. A solution that only adds branches, flags, helpers, or UI glue is suspicious; prefer removing stale paths, collapsing duplicated state, or moving behavior to the owning layer before adding code.
- Do not add code comments until they are genuinely needed to explain specific behavior the code itself cannot describe.
- Keep concrete implementations focused and interfaces small.
- Put behavior in the layer that owns the concept. Shared transcript/message shapes belong in storage or a dedicated shared package, not copied through server, ACP, and UI paths.
- Keep provider-facing data separate from display/transcript data. Do not mutate prompts and then repair snapshots by string matching; carry explicit typed boundaries instead.
- Native coding-agent parity is a hard invariant. Providers own model metadata, prompt construction, compaction, context limits, tools, and auth semantics; Jaz adapters may translate protocol shapes but must not override, duplicate, guess, or silently degrade them. Any intentional divergence must be explicit, bounded, measurable, and opt-in.
- Before releasing changes to ACP/native agent boundaries, compare Jaz against the corresponding native CLI for model identity and context window, first-turn prompt count/size, reload behavior, compaction, tool capabilities, and authentication. Missing provider metadata must remain unknown rather than falling back to a guessed value.
- Architect the native Jaz agent behind protocol-shaped interfaces, modeled after MCP-style request/response/event contracts. Typed content blocks, tool calls/results, permissions, streaming updates, and capabilities should cross explicit interfaces instead of direct server/provider coupling.
- Keep native runtime behavior transport-neutral. Native, ACP, and future protocol adapters should share internal turn/session/tool contracts; protocol-specific code translates only at the boundary.
- Preserve migration paths to ACP or MCP-style protocols when adding agent features. Prefer capabilities and feature detection over hardcoded runtime branches, and do not hide native-only semantics inside prompts.
- Split files when a feature starts mixing transport, persistence, formatting, and UI concerns. Avoid pushing files toward 1k lines without a strong structural reason.
- Frontend shared hooks and lib code must not import component-owned types. Put cross-layer contracts in `lib`.
- Keep feature diffs scoped. Do not mix unrelated UI polish, settings work, dependency churn, or generated output into behavioral changes.
- Prefer Viper's default field mapping. Add `mapstructure` tags only for real mismatches.
- Keep `main.go` files as command dispatch and process entrypoints only. Arbitrary domain types, helper functions, clients, transports, URL builders, and request/response shapes belong in the package that owns that concept; the only allowed exception is global Viper bootstrap/config wiring.
- Use Fx constructors directly in `fx.Provide`; avoid pass-through wrappers.
- Do not add defensive nil checks for required constructor-injected dependencies. If a required Fx service is missing, fail fast instead of silently degrading; model truly optional dependencies explicitly.
- Codex ACP defaults to the user's Codex OAuth credentials. Never silently pass coordinator provider keys to Codex subprocesses; a provider API key reaches Codex only when the user explicitly selects a non-OpenAI model provider (e.g. OpenRouter) for it.
- Target deployments run the Jaz server on a VM and clients on user computers; never assume client-local file paths are visible to the server or agents.
- Before handing off a completed feature or fix, run a code-review pass; use `thermo-nuclear-code-quality-review` when available.
- Every test you add must be useful: it must run in the relevant verification path and either protect real behavior or clarify a tricky contract. A test that is skipped, does not run, or provides no useful signal must not exist just to raise coverage.
- Reference repos (`openclaw`, `hermes`) are learning material, not authority.

## Backend Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gluonfield/jaz](https://github.com/gluonfield/jaz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
