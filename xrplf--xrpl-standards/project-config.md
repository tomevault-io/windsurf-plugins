---
trigger: always_on
description: This repository is the canonical home for **XRP Ledger Standards (XLSes)** — specifications and standards that govern the XRP Ledger ecosystem. Each XLS is a living document covering protocol amendments, system-level changes, or community/off-chain conventions. The full process is defined in [XLS-0001](../XLS-0001-xls-process/README.md) and summarized in [CONTRIBUTING.md](../CONTRIBUTING.md).
---

# Copilot Cloud Agent Instructions — XRPL-Standards

## Repository Purpose

This repository is the canonical home for **XRP Ledger Standards (XLSes)** — specifications and standards that govern the XRP Ledger ecosystem. Each XLS is a living document covering protocol amendments, system-level changes, or community/off-chain conventions. The full process is defined in [XLS-0001](../XLS-0001-xls-process/README.md) and summarized in [CONTRIBUTING.md](../CONTRIBUTING.md).

---

## Repository Layout

```
/
├── XLS-NNNN-slug/          # One folder per XLS spec (4-digit zero-padded number + hyphenated title)
│   └── README.md           # The XLS document itself (required)
├── templates/
│   ├── XLS_TEMPLATE.md     # Preamble + section scaffold for all new XLSes
│   └── AMENDMENT_TEMPLATE.md  # Extra scaffold for Amendment-category XLSes (Section 3 onward)
├── scripts/
│   ├── requirements.txt    # Python deps (install with: pip install -r scripts/requirements.txt)
│   ├── xls_parser.py       # Parses XLS README.md files and validates preamble metadata
│   ├── validate_xls_template.py  # Validates XLS structure against templates (Beta CI)
│   └── build_site.py       # Builds the GitHub Pages static site from XLS docs
├── CONTRIBUTING.md         # How to contribute (summarises XLS-1)
└── .github/
    ├── pull_request_template.md
    ├── workflows/           # CI workflows (see below)
    └── scripts/             # Scripts used by CI (assign_xls_number.py, etc.)
```

---

## XLS Document Format

Every XLS lives at `XLS-NNNN-slug/README.md` and **must** start with an RFC-822-style `<pre>` preamble block:

```
<pre>
  xls: [number]
  title: [max 44 chars, no "XLS" prefix]
  description: [max 140 chars, one sentence]
  author: Name (@github-handle), Other Name <email@example.com>
  category: [Amendment | System | Ecosystem | Meta]
  status: [Draft | Final | Living | Deprecated | Stagnant | Withdrawn]
  proposal-from: https://github.com/XRPLF/XRPL-Standards/discussions/NNN
  created: YYYY-MM-DD
  updated: YYYY-MM-DD          # optional
  implementation: [url]        # optional, for Amendment/System XLSes
  requires: [xls numbers]      # optional
  withdrawal-reason: [reason]  # required only when status is Withdrawn
</pre>
```

**Required sections** (all XLSes): Abstract, Rationale, Security Considerations.
**Amendment XLSes** additionally require the sub-structure from `AMENDMENT_TEMPLATE.md` covering STypes, Ledger Entries, Transactions, Permissions, and RPCs as applicable.

### Categories

| Category    | Description                                                             |
| ----------- | ----------------------------------------------------------------------- |
| `Amendment` | Requires an XRP Ledger amendment (on-chain protocol change via rippled) |
| `System`    | Affects XRPL protocol behavior (RPCs, P2P) but no amendment needed      |
| `Ecosystem` | Off-chain/community standards (metadata, registries, etc.)              |
| `Meta`      | Standards about the XLS process itself                                  |

### Statuses

`Idea` → `Proposal` → `Draft` → `Final` or `Living`

Also: `Deprecated` (Final XLS no longer recommended), `Stagnant` (Draft inactive ≥6 months), `Withdrawn` (removed by author — number cannot be reused).

---

## How to Add a New XLS

1. Start in GitHub Discussions (Idea or Proposal stage) — do **not** open a PR until there is community feedback.
2. Create a directory named `XLS-draft-<short-title>/` (agents/authors must NOT self-assign numbers for XLS numbers > 95).
3. Copy `templates/XLS_TEMPLATE.md` to `XLS-draft-<short-title>/README.md` and fill it in.
4. Open a PR. CI will assign the official XLS number automatically after a maintainer with write access approves; the `assign-xls-number.yml` workflow renames the directory and updates the preamble. Authors should not assign their own XLS number.

---

## CI Workflows

| Workflow                      | Trigger             | What it does                                                                                            |
| ----------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------- |
| `validate-xls.yml`            | PRs, push to master | Runs `python scripts/xls_parser.py` — validates preamble of **all** XLS docs                            |
| `validate-xls.yml` (beta job) | PRs                 | Runs `python scripts/validate_xls_template.py <changed files>` — checks section structure               |
| `assign-xls-number.yml`       | PRs                 | Detects `XLS-draft-*` directories and assigns the next available XLS number after write-access approval |
| `pre-commit.yml`              | PRs, push to master | Runs pre-commit hooks (trailing whitespace, end-of-file, prettier)                                      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XRPLF/XRPL-Standards](https://github.com/XRPLF/XRPL-Standards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
