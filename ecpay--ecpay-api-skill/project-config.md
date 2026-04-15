---
trigger: always_on
description: This is an **AI Skill repository** (Markdown knowledge base), not a traditional software project. There is no build system, no package manager, and no application code. All content is Markdown files consumed by AI coding assistants.
---

# Copilot Instructions — ECPay API Skill

This is an **AI Skill repository** (Markdown knowledge base), not a traditional software project. There is no build system, no package manager, and no application code. All content is Markdown files consumed by AI coding assistants.

## Architecture

Three-layer knowledge system:

- **`SKILL.md`** — AI entry point: decision trees, navigation, safety rules, test accounts. Routes queries to the correct guide + reference.
- **`guides/`** (28 files, indexed 00-24 + 02a/02b/02c) — Static integration knowledge with SNAPSHOT parameter tables. Provides process logic, caveats, code examples in 12 languages.
- **`references/`** (19 files, 431 URLs) — Real-time API spec gateway. Each file contains organized URLs pointing to `developers.ecpay.com.tw`. AI uses `web_fetch` on these URLs to get the latest official parameter specs before generating code.

The critical pattern: **guides/ tells you HOW to integrate; references/ gives you the CURRENT spec to integrate against.** Guides are sufficient for prototyping and initial development (parameter stability >95%). For production deployment or when API behavior doesn't match expectations, use `web_fetch` on references/ URLs to verify the latest specs.

### Four HTTP Protocol Modes

| Mode | Auth | Format | Services |
|------|------|--------|----------|
| CMV-SHA256 | CheckMacValue + SHA256 | Form POST | AIO payment |
| AES-JSON | AES-128-CBC | JSON POST | ECPG, invoices, logistics v2 |
| AES-JSON + CMV | AES-128-CBC + CheckMacValue (SHA256) | JSON POST | E-tickets (CMV formula differs from AIO) |
| CMV-MD5 | CheckMacValue + MD5 | Form POST | Domestic logistics |

### Supporting Files

- **`SKILL_OPENAI.md`** — Condensed version of SKILL.md for ChatGPT GPTs (custom GPT). Upload to GPT Builder Knowledge. When both exist, SKILL_OPENAI.md takes precedence for GPT platforms.
- **`SETUP.md`** — Unified installation guide for Codex CLI, Gemini CLI, OpenClaw, and ChatGPT GPTs. Contains Knowledge Files list for GPTs.
- **`AGENTS.md`** — Condensed entry point for OpenAI Codex CLI. Contains role, decision tree, critical rules, and test accounts.
- **`GEMINI.md`** — Condensed entry point for Google Gemini CLI. Mirrors AGENTS.md with Gemini-specific notes (Google Search instead of web_fetch).
- **`commands/`** (6 files) — Claude Code slash commands. Navigation only, ≤20 lines each. Do not duplicate SKILL.md logic.
- **`test-vectors/`** — Deterministic test vectors for CheckMacValue (SHA256/MD5), AES encryption, and URL encoding comparison. Run `python test-vectors/verify.py` (requires `pycryptodome`) to validate all 18 vectors.
- **`scripts/SDK_PHP/`** — Official ECPay PHP SDK with 134 verified examples. Read-only reference; do not modify.

## Validation Commands

```bash
# Validate AI Section Index line numbers in guides/13, 14, 24
bash scripts/validate-ai-index.sh

# Validate version sync across all 9 platform entry files
bash scripts/validate-version-sync.sh

# Validate AGENTS.md ↔ GEMINI.md invariant sections (decision tree, critical rules, test accounts)
bash scripts/validate-agents-parity.sh

# Validate all internal cross-guide links (no broken references)
bash scripts/validate-internal-links.sh

# Verify all 18 test vectors (CheckMacValue, AES, URL encoding)
pip install pycryptodome && python test-vectors/verify.py

# Manually trigger URL check (also runs weekly via GitHub Actions)
# Go to Actions tab → "Validate Reference URLs" → Run workflow
```

> **macOS note**: `validate-ai-index.sh` and URL-check scripts use `grep -P` (Perl regex), which is not supported by macOS BSD grep. Install GNU grep first: `brew install grep` and ensure `ggrep` is on your PATH, or run via CI (ubuntu-latest).

CI runs automatically:
- **`validate.yml`**: On PRs and main-branch pushes that modify any guide or script file, `test-vectors/`, or platform entry files (SKILL.md, AGENTS.md, etc.); runs all four validation scripts + `python test-vectors/verify.py`
- **`validate-references.yml`**: Weekly Monday 02:00 UTC + manual trigger

## Key Conventions

### Version Sync (mandatory)

When changing the version number, update ALL of these:
- `SKILL.md` front-matter `version` field (line 3)
- `SKILL_OPENAI.md` version reference (line 3)
- `README.md` version string (line 5)
- `SETUP.md` version reference (line 3)
- `AGENTS.md` version reference (line 3)
- `GEMINI.md` version reference (line 3)
- `.github/copilot-instructions.md` version reference (line 75)
- `../CLAUDE.md` version and guide count (root repo instructions)
- `CONTRIBUTING.md` version reference (English summary section)

`validate-version-sync.sh` checks the canonical version pattern from SKILL.md front-matter (e.g. `V2.7`). `CHANGELOG.md` uses three-segment semver (`1.0.0`) per Keep a Changelog convention — the entry-point files and CHANGELOG may use the same or different number of segments; both must be updated when releasing.

### Platform Entry File Parity


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ECPay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
