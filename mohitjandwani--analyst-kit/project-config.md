---
trigger: always_on
description: **Analyst Kit is not an application — it is a packager and distributor of *skills* for
---

# Analyst Kit — equity-research skills for AI agents

**Analyst Kit is not an application — it is a packager and distributor of *skills* for
AI coding agents** (Claude Code, Codex, and any runtime that can read a folder of
instructions). A "skill" is a self-contained folder under `skills/<name>/` whose
`SKILL.md` holds agent instructions, optionally alongside runnable `scripts/`,
`references/`, `templates/`, and `assets/`. The skills are the product; the Node code
under `src/` is plumbing that catalogs, resolves, and installs them.

There are two ways to consume the skills:

1. **Claude Code / Cowork plugin** — add the marketplace, install a persona plugin.
2. **Node installer** (`bin/analyst-kit.js` → `src/`) — copy skills into any runtime.

This file is the entry point for an AI agent working in this repo. If you only want to
*use* the skills, read **[Quick start](#quick-start--install-the-skills)**. If you are
*modifying* the repo, read **[Architecture](#architecture--how-a-skill-becomes-installable)**
onward. See also [`README.md`](README.md) (end-user docs), [`CLAUDE.md`](CLAUDE.md)
(contributor rules), and [`compatibility.md`](compatibility.md) (per-runtime behavior).

---

## Quick start — install the skills

### Option A · Marketplace plugin (primary — no clone, no Node)

Claude Code and Claude Cowork install the **single `analyst-kit` plugin** from the
same marketplace. It bundles every skill, the `research-auditor` subagent, and a
SessionStart runtime hook.

```
/plugin marketplace add mohitjandwani/analyst-kit
/plugin install analyst-kit@analyst-kit
```

In **Cowork** (desktop app), no terminal:

1. **Customize → Plugins → Add ▾ (top right) → Add marketplace.**

   ![Cowork — Plugins panel: Add ▾ → Add marketplace](docs/images/cowork-1-add-marketplace.png)

2. In **Add marketplace**, enter and select your repo: `mohitjandwani/analyst-kit`.

   ![Cowork — Add marketplace: enter mohitjandwani/analyst-kit](docs/images/cowork-2-enter-repo.png)

3. Add the **analyst-kit** plugin, then enable **Settings → Capabilities → Code execution**.

### Option B · Node installer (any runtime)

One command installs *all* skills into a runtime and wires them into the agent's
system/common prompt. Needs only **Node ≥ 18**:

```bash
npx github:mohitjandwani/analyst-kit claude-code     # or: codex · openclaw · cowork
```

From a clone, the same flows run through the bundled CLI (the package isn't published to npm
yet — the name `analyst-kit` is available — so invoke the file directly, not `npx analyst-kit`):

```bash
node bin/analyst-kit.js list                                              # browse skills + personas
node bin/analyst-kit.js claude-code                                       # install ALL skills (user scope)
node bin/analyst-kit.js install <skill|persona> --platform claude-code    # install one + its deps
node bin/analyst-kit.js install <skill> --platform codex --scope project --dry-run
node bin/analyst-kit.js doctor --platform claude-code                     # check runtimes + API keys
node bin/analyst-kit.js uninstall <skill|persona> --platform claude-code
```

Add `--scope project` to install into the current project (`./.claude/skills`, …)
instead of your home directory.

### API keys

The plugin's `userConfig` prompts for keys at enable time and stores them in the OS
keychain; the SessionStart hook bridges each non-empty value into `$AK_HOME/.env`
(which the per-skill preamble sources), so scripts read `os.environ[...]` unchanged.
**`FMP_API_KEY` is required**; `FINMIND_TOKEN` and `SERPAPI_API_KEY` are optional
(add them in the config UI anytime, or let the per-skill onboarding ask when the
skill runs and soft-disable until provided). On Codex/Node the onboarding prompts and
writes the same `.env`.

| Variable | Used by | Required | Get it |
|----------|---------|----------|--------|
| `FMP_API_KEY` | `financialmodellingprep`, `company-wiki`, `company-universe-manager` | **Yes** — plugin config | <https://site.financialmodelingprep.com/developer/docs> |
| `FINMIND_TOKEN` | `finmind` | Optional (Taiwan) | <https://finmindtrade.com/> (free) |
| `SERPAPI_API_KEY` | `market-intelligence` | Optional (Google Trends) | <https://serpapi.com/> (free tier: 100/mo) |
| `SEC_EDGAR_UA` | `sec-filings`, `13f-analysis` | Auto-generated | — |

`13f-analysis` and `sec-filings` read SEC EDGAR directly with no key — `SEC_EDGAR_UA`
is auto-generated per install from a one-time user id (`analyst-kit-setup
ensure-identity`; `edgar.py` derives the UA from it). Skills that run code bootstrap
their own dependencies on first use; **Python** and **Bun** are per-skill
prerequisites the installer does not install for you.

### Verify it loaded

Ask a trigger phrase — e.g. *"deep dive on NVDA"* — and the matching skill loads.
Agents discover skills by `name` + `description` only, so triggers live in the
description's `Triggers:` clause.

---

## Available skills

18 skills split into **capabilities** (one atomic job — a data source, an engine, a
deliverable, or reusable knowledge) and **workflows** (an engagement entry point that
orchestrates capabilities via `requires:`). Every skill depends on the
`analyst-kit-core` runtime (auto-installed); the **Needs** column lists *additional*

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mohitjandwani/analyst-kit](https://github.com/mohitjandwani/analyst-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
