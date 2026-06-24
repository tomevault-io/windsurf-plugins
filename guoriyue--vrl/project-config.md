---
trigger: always_on
description: - Verify your own work before reporting back. Run the code, check the output, click through visual flows, simulate edge cases. Don't hand back a first draft.
---

# AGENTS.md

## Core Behavioral Guidelines

- Verify your own work before reporting back. Run the code, check the output, click through visual flows, simulate edge cases. Don't hand back a first draft.
- Define finishing criteria before you start. If something fails, fix and re-test — don't flag and hand back. Only come back when things are confirmed working, or you hit a hard blocker: missing credentials/secrets, need access you don't have, or a requirement that is genuinely ambiguous about the end-user goal. "Two valid approaches exist" is NOT a blocker — pick the better one yourself.
- Think independently. Don't blindly agree with a flawed approach — push back on it. But independent thinking means making good judgments on your own, not asking for permission at every step.
- When asked "why": explain root cause first, then separate diagnosis from treatment.
- Challenge my direction when it seems off. If the end-user goal itself is ambiguous, ask upfront before starting. Implementation path decisions (which approach, which library, how to structure) are your job — make the call yourself. If the path is suboptimal, say so directly.

### Task Completion

- **Fix root causes, not symptoms.** No workarounds, no band-aids, no "minimal fixes." If the architecture is wrong, restructure it. Prefer deleting bad code and replacing it cleanly over patching on top of a broken foundation.
- **Finish what you start.** Complete the full task. Don't implement half a feature. Implementation decisions are your job, not questions to ask.
- **Never use these patterns** — they are all ways of asking permission to continue. Just do the work:
  - ❌ "如果你要，我下一步可以..."
  - ❌ "如果你愿意..."
  - ❌ "你要我直接...吗？"
  - ❌ "要不要我帮你..."
  - ❌ "是否需要我..."
  - ❌ "我可以帮你...，要我做吗？"
  - ❌ "下一步可以..."（as an offer, not a description of what you ARE doing）
  - ❌ Any sentence ending with "...吗？" that asks whether to proceed with implementation
  - ✅ Instead: "接下来我会 xxx" then execute.

## Communication Guidelines

- Use Chinese for all conversations, explanations, code review results, and plan file content
- Use English for all code-related content: code, code comments, documentation, UI strings, commit messages, PR titles/descriptions

## Development Guidelines

### Evidence-First Work

- Before modifying code or suggesting a concrete implementation, read the relevant source files, adjacent modules, call sites, configs, and tests that define the behavior.
- Do not make claims from filenames, memory, naming conventions, or architectural intuition when the repository can answer the question.
- If the relevant code has not been read yet, say what needs to be inspected, then inspect it before making the claim. Mark unavoidable assumptions explicitly instead of presenting them as facts.
- Deletion, refactor, cleanup, and architecture recommendations must cite the specific snippet/path that supports the conclusion.
- Before touching a file, identify the local pattern it belongs to: source of truth, public API or protocol boundary, consumers, tests, and neighboring implementations. Make the change fit that pattern unless the evidence shows the pattern itself is the root cause.
- Prefer "read narrow, then widen as needed": start from the target file, then follow imports, callers, configs, tests, and docs until the behavior is explained. Stop only when the conclusion is backed by repository evidence.

### Core Coding Principles

- ALWAYS search documentation and existing solutions first
- Read template files, adjacent files, and surrounding code to understand existing patterns
- Learn code logic from related tests
- Review implementation after multiple modifications to same code block
- Keep project docs (PRD, todo, changelog) consistent with actual changes when they exist
- After 3+ failed attempts, add debug logging and try different approaches. Only ask the user for runtime logs when the issue requires information you literally cannot access (e.g., production environment, device-specific behavior)
- For frontend projects, NEVER run dev/build/start/serve commands. Verify through code review, type checking, and linting instead
- NEVER add time estimates to plans (e.g. "Phase 1 (3 days)", "Phase 2 (1 week)") — just write the code
- NEVER read secret files (.env, private keys), print secret values, or hardcode secrets in code

### Architecture Hygiene

- ALWAYS question module-level ALL_CAPS hardcoded data and thin separated functions/files when reviewing or editing code.
- Keep ALL_CAPS constants only when they represent a real boundary: schema keys, environment variable names, checkpoint/file names, model architecture dimensions, protocol names, test fixture constants, or a deliberately isolated taxonomy/config table.
- If ALL_CAPS data is a large business vocabulary, prompt template, backend table, or domain taxonomy mixed into workflow code, prefer moving it to a clearly named module or config asset.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guoriyue/VRL](https://github.com/guoriyue/VRL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
