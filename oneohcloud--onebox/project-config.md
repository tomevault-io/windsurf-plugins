---
trigger: always_on
description: **Table of contents**:
---

# OneBox — Project Notes for Claude

## Reading guide

**Table of contents**:

1. **User-visible text** — UI copy, toasts, CHANGELOG; never say "订阅" / "subscription"
2. **Hashed domain allowlist is a secret** — never write the pre-image in code, comments, docs, commits, or logs
3. **Reading third-party source** — clone upstream at the pinned version, don't rely on web search
4. **Deep link bug triage** — log-first checklist (Rust saw it? hot/cold? parsed? timing?)
5. **Logging discipline** — write logs anticipating triage (companion to §4)
6. **Release workflow triggers** — one workflow, four channels, `make bump` only
7. **Privileged helper version bump (macOS)** — editing `src-tauri/helper/` requires a manual `CFBundleVersion` bump; no auto-sync
8. **GitHub CLI access** — `gh` is the preferred interface for this repo
9. **Test-first cadence** — write fn → write test → run local → pass → next; CI runs both layers
10. **Verifying Linux from a macOS host** — `make linux-check`; commits are not transport
11. **Workflows that need my hands** — `scripts/tmp-*.sh` with manual gates + sanity checks
12. **Design Philosophy** — principles driving DNS / template subsystems
13. **Windows Platform Implementation Philosophy** — native Win32 over PowerShell
14. **Step-by-step semantic analysis** — expand every verb in a sequence before inserting adjacent to it
15. **Subsystem deep-dives** — DNS override, config templates, update argv suppression (in `docs/claude/`)

### Meta-rule: when rules tension against each other

**User-visible expression chases current; internal code identifiers chase stable.**

Concrete case: UI copy must move from `订阅` to `配置`, but the function `addSubscription()` stays — because renaming it touches dozens of call sites for zero user-facing benefit.

When in doubt: ask "what breaks if I change this?" If only a reader's eye is involved, follow the newer rule. If call sites or external references break, preserve the old shape.

### Meta-rule: what belongs in this file

**Belongs here**:
- Constraints re-learned through repeated pain (a bug root-caused the same way twice).
- Design decisions whose rationale is invisible from the code alone — especially *why we DON'T* have some plausible mechanism.
- Cross-platform or cross-subsystem principles that need one canonical statement.
- Workflow invariants the assistant can't derive from the repo (manual-gate procedures, VM hostnames, which command wraps what).

**Does NOT belong**:
- Single-point implementation details — a code comment is enough.
- Temporary workarounds — use a `TODO` at the site.
- Style preferences with no downstream consequence.
- Anything `rg` + a minute of reading would turn up reliably.
- Detailed subsystem walk-throughs — put those in `docs/claude/<name>.md` and link from the "Subsystem deep-dives" section.
- Rules that can be encoded as tooling — move the rule into the tool and delete the prose. Example: the "canonical Tailwind classes over arbitrary px" policy used to live here; it now lives in `scripts/check-tailwind-canonical.ts` + a `.husky/pre-commit` gate, so this doc no longer carries it.

Rule of thumb for sedimenting a new entry: if you catch yourself explaining the same thing across three different conversations, write it down. If it's a one-off, don't — this file's value is inverse to its length.

---

## User-visible text: shortest, plainest, product-accurate

Meta-rule: **whenever the user will read the string, use the shortest product-accurate wording — avoid developer jargon, implementation details, and SaaS/billing vocabulary that doesn't fit what OneBox actually is.** This applies to every user-facing surface: UI copy, toast/alert text, placeholders, labels, button captions, aria-labels, list fallback names, and `CHANGELOG.MD`.

### Never say "subscription" / "订阅"

Use **"配置"** (Chinese) and **"Config"** (English) when referring to the user's saved server configurations. Avoid every variant of:

- `订阅` / `订阅管理` / `订阅列表` / `订阅链接` / `订阅文件`
- `配置文件` (use the shorter `配置` instead)
- `subscription` / `subscriptions` / `Subscription(s)`

Shortest possible term wins: `配置` (2 chars) beats `配置文件` (4 chars) beats `订阅配置` (4 chars). English: `Config` beats `Configuration` beats `Subscription`.

Why: it's a local config store, not a SaaS service — "subscription" misleads users into expecting billing / account flows that don't exist.

**Boundary**: this only governs **display text**. Code identifiers (`addSubscription`, `GET_SUBSCRIPTIONS_LIST_SWR_KEY`, i18n keys like `add_subscription`) keep legacy names to avoid a disruptive rename.

### CHANGELOG entries

`CHANGELOG.MD` is written for **end users**, not developers. Each entry should be a single sentence describing what the user can observe. Do not include implementation details, file paths, config field names, code-level terms (e.g. `route_exclude_address`, `inbound`, `hijack-dns`), root-cause analysis, RFC terminology, or emoji. Provide both English and Simplified Chinese entries.

Bad: `Fixed bypass-router mode where the Mixed inbound listened on 127.0.0.1, making LAN hosts unreachable`
Good: `Fixed bypass-router mode not handling DNS and traffic from other devices on the LAN`

## Hashed domain allowlist is a secret

The compile-time hash lists (`KNOWN_HOST_SHA256_LIST` in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OneOhCloud/OneBox](https://github.com/OneOhCloud/OneBox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
