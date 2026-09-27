---
trigger: always_on
description: - Keep the core product limited to static, single-image text CAPTCHA recognition for digits, English letters, alphanumeric strings, and one-step integer arithmetic.
---

# Project Instructions

## CAPTCHA scope

- Keep the core product limited to static, single-image text CAPTCHA recognition for digits, English letters, alphanumeric strings, and one-step integer arithmetic.
- Prefer common CAPTCHA formats. Do not expand the core model for niche scripts, multi-step mathematics, image selection, sliders, puzzles, animation, or behavioral challenges without an explicit product decision.
- Arithmetic supports `+`, `-`, `*`, `/`, `x`, `X`, `×`, and `÷`. Training data should use nonnegative subtraction and exact division. Common suffix forms such as `=?`, `=`, `?`, and no suffix must be represented.

## Model and data changes

- Treat the repository benchmark, not an individual screenshot or same-generator random split, as the source of truth for model quality.
- Never add benchmark fixtures to training or validation data. Split synthetic templates, public dataset generators, and real website families by `group`; one group must not cross splits.
- Every training image must have a stable label, SHA-256, source, license id, and scenario/template group in the dataset manifest.
- Do not train on or redistribute a public dataset until its license and provenance have been recorded. A repository license alone does not automatically license bundled images.
- Keep downloads, extracted third-party data, checkpoints, and training output out of git. Commit only manifests, generation code, pinned metadata, documentation, and intentionally redistributable fixtures.
- A production model replacement requires reproducible training inputs, a completed model card, Paddle-to-ONNX consistency checks, the frozen benchmark, and Chrome/Edge offline verification. Do not replace `public/models` from a single issue sample.

## Open-source scenario contributions

- Secondary TODO: maintain a concise contributor retraining guide as the model and dataset recipe stabilizes; this documentation must not block current model-quality and browser-verification work.
- Convert requests for a new CAPTCHA style into a reproducible scenario contribution: authorized samples, exact labels, license/provenance, a unique group, a failing held-out benchmark, and a description of the visual mechanism that current coverage misses.
- Prefer extending deterministic generators or augmentation families over adding site-specific image hacks. Keep site-specific behavior behind an explicit per-site mode only when a general model/decoder change is not justified.
- Update `docs/model-training.md`, the dataset source catalog, tests, and the model card whenever the training recipe, supported scope, charset, data policy, or release gate changes.
- Preserve the single-model-first architecture. Add a second production model only after the primary model fails the documented selective-accuracy and coverage gates on isolated groups.

## Release gates

- Optimize for selective precision: automatic-fill precision must be at least 99.5%, coverage at least 80%, cold start at most 3 seconds, and warm single-image P95 at most 500 ms on the documented reference machine.
- A missing or weak arithmetic operator is not evidence for a digits fallback. Abstain instead of automatically filling a structurally ambiguous value.
- Report results by category, source, scenario group, and arithmetic symbol. Aggregate accuracy alone is insufficient.

## Release experience

- Every version update opens its standalone “本次更新” page once. Keep a “新功能” entry in the popup header so users can reopen it at any time.
- First-install setup and every release introduction start with a concise welcome scene, followed by explanations or settings reached through native scrolling. Use shared glass surfaces and elastic scene transitions, with reduced-motion support. Release introductions describe shipped benefits and end with a “开始使用” action.
- Every published version, including patch releases, must have its own dated introduction and bilingual user-facing notes in the local release archive. Keep historical entries accurate to what that version shipped. Read the full notes from `CHANGELOG.md`; maintain the shared brand rules in `DESIGN.md`. Historical browsing must preserve the installed-version reminder and user settings.
- Apply the same editorial standard to release headlines, summaries, introductions, full local notes, and GitHub Release notes: explain a shipped feature, a useful interaction improvement, an affected browser or website, a concrete fix, or a permission/privacy change the user needs to understand. Lead with the user's task and the resulting behavior; include technical terms only when needed to act or troubleshoot.
- Keep developer-support promotions, donation-page additions, code restructuring, dependency/build/test changes, training implementation details, and maintainer workflows in commits or developer documentation. When such work has a verified user-facing effect, describe that effect in the release notes. Scale the introduction to the actual change; a small fix can have one highlight. A maintenance-only version still gets a brief, factual introduction about the continuity of use, without invented feature, reliability, or performance claims.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hongshuo-wang/local-captcha-solver](https://github.com/hongshuo-wang/local-captcha-solver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
