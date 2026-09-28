---
trigger: always_on
description: Universal **OCP** compatibility **bridge** monorepo. The stack is complete and published — no phased MVP cuts.
---

# AGENTS.md — opencode-plugin-compat

## Product

Universal **OCP** compatibility **bridge** monorepo. The stack is complete and published — no phased MVP cuts.

- Goal: **any OpenCode plugin runs unchanged** on cooperating hosts.
- OCP is an **external compatibility layer** for MiMo/Kilo/OpenCode, the Pi family, and DeepSeek Harness — hosts are read-only references; all OCP work lives in this repo.

### Non-negotiable vendor boundary

OCP is a plugin compatibility layer. **Patching a host shipped by another vendor defeats the product and is not an implementation option.**

- **Never propose, plan, attempt, or make a change to a host checkout, host package, installed host binary, or vendor source as part of OCP work.** Do not treat a vendor patch as a fallback, experiment, diagnostic shortcut, prerequisite, or possible solution. This prohibition applies even when a host source checkout is available locally and even when changing it would be technically easy.
- MiMo, Kilo, OpenCode, pi, omp, and deepseek-harness source trees are **read-only evidence only**: use them to understand contracts and runtime behavior. No OCP task may leave modifications in those repositories.
- **Every compatibility fix must live entirely in OCP** and work against the stock installed host through public or host-injected extension/runtime seams. For Pi-family hosts, use `pi-bridge`, `ExtensionAPI`, `ExtensionAPI.pi`, live objects intentionally exposed through that runtime, extension UI, and advertised tools. For DSH, use `dsh-bridge`, Cordis `LlmAdapter` / `ctx.llm.registerAdapter`, `ctx.credentials`, and `cordis.patch.yml`.
- Private host internals may be inspected to understand behavior, but they are not an implementation surface. If the installed host's extension/runtime seams cannot express the required behavior, stop and report a concrete compatibility limitation. Do not cross the vendor boundary to make the limitation disappear.
- Before designing a fix, inspect the actual executable, resolved package, plugin symlink/install tree, and loaded `dist` entry used by the reproduction. A nearby host checkout is never presumed to be loaded and never becomes a writable dependency of the solution.
- **Interactive behavior requires an interactive acceptance test.** Unit/type tests are supporting checks, not proof. For mode switches, approval dialogs, plan review, or execution handoffs, rebuild/install OCP and the consumer provider, start the real stock host in a TTY, drive the complete user flow, and verify the host transcript, side effect, and absence of retries before saying the issue is fixed.
- These rules apply to the main agent and every delegated subagent. Prompts to subagents must identify host repositories as read-only and forbid edits there.

### Non-negotiable OCP / consumer-provider boundary

The dependency direction is one-way: **OCP may know consumer providers; consumer providers must not know OCP.** OCP must adapt published providers without requiring OCP imports, host branches, or fork vocabulary in their source.

- **Provider stays unchanged:** never solve compatibility by adding `@opencode-compat/*`, OCP detection, fork environment variables, alternate tool names, or host-specific schemas to a consumer provider. Provider-side structural capability contracts must be host-neutral; OCP installs them before provider load.
- **Generic core, optional integrations:** generic facade, adapter, loader, config, and stream translation code must work for arbitrary OpenCode/AI-SDK providers. A provider-specific integration (currently Cursor host tools) must live in a clearly named optional module, activate only for an explicit package match, fail open when that provider is absent, and never become a prerequisite for generic providers.
- **OCP owns translation end-to-end:** fork paths, canonical tool catalog translation, call inputs, results, prompt/history replay, opaque resume ids, schemas, MCP/resource vocabulary, agents, and mode semantics are OCP responsibilities. The provider must see canonical OpenCode shapes.
- **No translation residue in providers:** if OCP normalizes an alternate host tool, schema, result envelope, approval outcome, or history record, remove the corresponding host parser and identity from the consumer provider in the same change. Never preserve provider-side fallbacks “just in case”; that recreates two contracts and makes the provider host-aware. Add provider architecture tests before considering relocation complete.
- **Provider identities stop at OCP:** provider-specific OCP behavior must be selected by an explicit package match and live in a clearly named optional module. Generic adapters receive only neutral callbacks/data and must not contain `cursorIntegration`, Devin branches, provider package imports, or sibling-provider schemas. Consumer providers must never export aliases for one another.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oakimov/opencode-plugin-compat](https://github.com/oakimov/opencode-plugin-compat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
