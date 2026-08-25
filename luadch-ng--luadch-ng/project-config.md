---
trigger: always_on
description: Context for Claude Code (and any AI assistant) working on luadch. Read this before
---

# CLAUDE.md

Context for Claude Code (and any AI assistant) working on luadch. Read this before
making changes — it captures the working agreement, architecture map, and roadmap
that span sessions. Engineering how-to (core-module authoring, testing contract,
security checklists, Definition of Done) lives in
[`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md).

User communication is in **German**; all written artifacts (this file, code, comments,
commits, PRs, issues) stay in **English** so other contributors can read them.

---

## 1. Working agreement (non-negotiable)

These rules are set by the maintainer and apply to every change.

### 1a. Per-change discipline (every PR, no size exemption)

1. **Security and consistency come first.** Treat any change touching network I/O,
   authentication, ADC protocol parsing, or configuration as security-sensitive. When
   fixing a pattern in one place, grep for the same pattern across the repo and fix it
   everywhere - divergent code paths are a defect.
2. **No spaghetti code.** Prefer small, focused functions and modules. Don't grow
   `core/cfg.lua` or `core/hub.lua` further; if new logic doesn't have an obvious home,
   propose a new module before writing the code.
3. **Deep-dive before implementation.** Analyse the issue/idea from the source
   outward before writing code, even when it costs more tokens. A clean
   implementation pays the tokens back twice over (#179: the deep-dive replaced a
   proposed 50-LoC redesign with a 1-line fix).
4. **An issue/plan is a hypothesis, not ground truth.** Always re-derive the root
   cause from spec + current source before implementing. If the issue/plan is wrong,
   correct the issue/plan - do not implement the wrong thing. (HUBI, CTM/RCM, the
   #179 split-table proposal: same trap three times.)
5. **Verify every assumption** against the current code/spec before building on it.
   Recalled memory and old docs are point-in-time; confirm before relying.
6. **Mandatory two-pass pre-merge review.** Before any merge - regardless of how
   small the diff - run: (a) an independent reviewer (subagent / fresh perspective)
   and (b) a maintainer-side spot-check. The review covers **security**, **new
   bugs**, **breaking existing behaviour**, **consistency / anti-spaghetti**. No
   "it's just a one-liner" exemption: the #179 one-line fix's review is exactly what
   caught a latent counter underflow.
7. **Regression tests must provably fail pre-fix.** A test green on both old and new
   code proves nothing. For every fix, demonstrate the new test FAILS on the
   unpatched code and PASSES patched (PR #177 lesson; applied per-tier in #179).
8. **Small reviewable PRs.** One logical change per PR. Reference the GitHub issue it
   closes. Never bundle unrelated fixes.
9. **No wall of text.** Chat answers, issues, PR bodies, release notes: minimal,
   technical, complete - result first. Internal artifacts (commit messages, code
   comments, repo docs, phase journals) stay as detailed as needed.

### 1b. Phase & milestone discipline

10. **One phase at a time.** Work proceeds strictly phase by phase (see §5 Roadmap).
    Do not pull tickets forward from a later phase, even if they look trivial.
11. **Review gate between phases, and before any release/milestone.** Run an
    explicit, PR-grade review (same depth as 1a.6) covering:
    - **Security** - input validation, auth boundaries, network surface, file I/O.
    - **Consistency** - did similar code paths drift apart? Did naming get inconsistent?
    - **Code quality** - readability, dead code, duplication, function length.
    - **Build & smoke test** - both Linux and Windows builds succeed, hub starts, a
      test client (`adcs://127.0.0.1:5001`) can connect.
    - **Docs currency** - this file and the affected `docs/*.md` were updated in the
      same PR whenever architecture, conventions, module layout, or defaults changed.
      A stale CLAUDE.md poisons every future session's context; treat doc drift as a
      review-blocking defect, not cosmetics.
12. **Fix-then-advance.** Anything found in the review must be fixed before the next
    phase begins or the release is cut. No "we'll get back to it." If something is
    genuinely out of scope, open a tracking issue and link it from the phase summary.

When uncertain whether a change fits the current phase, **stop and ask the maintainer**.

---

## 2. Project overview

luadch is a DC++ **ADC** hub server written in Lua with a thin C launcher
(`hub/hub.c`) that embeds the Lua interpreter and hands off to `core/init.lua`.

- **Current source version:** `v3.2.0-dev` on `master`, `PROGRAM_NAME = "Luadch-NG"`
  (see `core/const.lua`). The 3.1.x maintenance line keeps `PROGRAM_NAME = "Luadch"`.
- **Latest release:** `v3.1.15` (2026-07-29, on `release/3.1.x`)
- **Status:** the Phase 1-7 modernisation programme is content-complete; work is now
  3.2.x feature development (Phase 8+) plus 3.1.x security-only maintenance
  until the `v3.2.0` tag (which EOLs the 3.1.x line - see §8).
- **Open issues:** check `gh issue list --repo luadch-ng/luadch-ng` (never trust a
  count written into a doc).
- **Testing:** a pure-Lua unit suite (`tests/unit/*_test.lua`) plus a protocol-level

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luadch-ng/luadch-ng](https://github.com/luadch-ng/luadch-ng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
