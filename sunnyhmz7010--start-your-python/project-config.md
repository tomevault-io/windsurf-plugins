---
trigger: always_on
description: These rules are written as the shared baseline for this project family.
---

# AGENTS.md

## Reusable Rules

These rules are written as the shared baseline for this project family.

- Keep the `Reusable Rules` block aligned across sibling repositories unless the user explicitly asks for a deliberate deviation.
- Treat reusable-rule updates as bidirectional synchronization: when shared rules change in one repository, apply the same block to sibling repositories in the same task.
- Keep repository-specific product, packaging, signing, route, architecture, and handoff details under `Repository-Specific Rules`, not in this section.

### General Working Style

- Prefer minimal, targeted changes over broad refactors.
- Preserve existing product copy unless the task requires rewriting it.
- Keep user-facing docs concise and practical; avoid adding AI collaboration notes or marketing filler unless explicitly requested.
- Keep public root `README.md` files user-facing and polished: lead with value, use concise feature/usage framing, include externally useful examples, and avoid internal progress notes, AI handoff notes, operational constraints, or release-process guidance.
- Prefer the project-family README pattern when applicable: centered logo/title/value summary, badges and primary links, a `---` divider, screenshot preview, capability breakdown, quick start, usage/integration examples, feature details, local development, security reporting, license, Star History, and footer credit.
- Keep README structure user-journey oriented: what it is, why it matters, what it can do, how to start, how to use or integrate it, then contributor-facing local development notes.
- Keep README prose tight. Group capabilities by user-facing surface or scenario, use concrete statements and copyable minimal examples, and avoid repeating the same capability unless new context is added.
- README license text, license badge, root `LICENSE`, package metadata, and build metadata must agree; update them together or explicitly call out intentional differences.
- Contributor rules, AI guidance, handoff notes, release conventions, missing-work notes, and local helper commands belong in `AGENTS.md`, not public docs. Do not create `docs/`, `notes/`, `tmp/`, or similar directories just to store that material unless the user asks.
- In public docs, write commands with standard upstream tooling rather than local wrappers, aliases, shell functions, or private helper commands.
- For searches, prefer `rg`.
- Use `apply_patch` for manual edits when the environment is stable.
- Do not run destructive git commands unless explicitly requested.

### Validation And Hygiene

- Keep the working tree clean before handoff: do not leave local build outputs, dependency caches, debugging screenshots, or temporary troubleshooting files committed or untracked.
- When the environment lacks a required toolchain and the user does not need full local verification, skip heavy verification only when necessary and say so explicitly.
- Release notes are user-facing change logs. Do not include internal verification/process statements such as having run tests, builds, audits, or CI checks unless explicitly requested.
- When repository structure, commands, external capabilities, release process, or recurring engineering pitfalls change, update `AGENTS.md` in the same task. Keeping this file current is required, not optional.
- If newly learned guidance appears reusable across repositories, ask whether to scan sibling `AGENTS.md` files, apply the shared rule, and push those updates.
- For GitHub-hosted repositories, maintain the baseline repository-governance files consistently across projects unless the user explicitly asks for divergence. This baseline includes `LICENSE`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, issue templates, and similar repo-health/community files.
- "Consistently" does not mean every line must be identical. Keep the structure, tone, and policy baseline aligned, but make the necessary project-specific substitutions for repository name, product name, links, version fields, platform fields, security scope, issue-form fields, and other repo-specific facts.
- If one of those GitHub governance files changes in a way that should become the new shared baseline, ask whether to propagate it across sibling GitHub repositories and push the updates while preserving required project-specific substitutions.

### Security And Review

- Review code with a bug-risk mindset first. Prioritize functional regressions, security issues, breaking changes, and missing tests before style or cleanup suggestions.
- If code returns `text/html` built from server-side string templates, HTML-escape all text fields from settings, persisted data, and user-controlled input before interpolating them into tags such as `<title>`, headings, attributes, or inline scripts.
- Do not assume only frontend `innerHTML` paths are XSS-relevant; also inspect backend-rendered HTML, email templates, CMS fragments, and raw string formatting that bypasses auto-escaping.
- For admin permission checks, prefer no-side-effect probes against real resources.
- Do not use invalid create requests to probe permissions; validation failures can mask the real authorization result and create misleading server logs.

### Dependency And Upgrade Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sunnyhmz7010/start-your-python](https://github.com/sunnyhmz7010/start-your-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
