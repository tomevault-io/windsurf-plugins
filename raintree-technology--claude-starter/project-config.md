---
trigger: always_on
description: This file provides guidance to Codex when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex when working with code in this repository.

---

## 1. What is this repository?

**Codex-starter** is a **template repository** - a production-ready starting point for building your own Codex configuration.

**Purpose:**
- Drop into any project to give Codex domain expertise
- 40+ pre-built skills across AI, APIs, blockchain, backend, frontend
- TOON format support for 30-60% token savings on large datasets
- Clean template for building your own skills and commands
- Documentation pulled separately (not included in repo)

**This is NOT:**
- An application to run
- A library or framework
- Code to import

**This IS:**
- Configuration for Codex
- A collection of expertise (skills)
- Token optimization tools (TOON)
- A template you can learn from and extend

---

## 2. Complete Documentation

For complete documentation, see:

- **[`.Codex/README.md`](.Codex/README.md)** - Quick start, installation, usage guide
- **[`.Codex/DIRECTORY.md`](.Codex/DIRECTORY.md)** - Complete reference documentation
- **[`/docs/`](docs/)** - How to build skills, commands, hooks

---

## 3. Directory Structure

```
Codex-starter/
├── docs/                # Template usage guides
│   ├── creating-components.md
│   ├── examples.md
│   └── FAQ.md
│
├── .Codex/             # Codex configuration
│   ├── skills/          # 40 auto-invoked skills
│   ├── commands/        # 7 slash commands
│   ├── hooks/           # 5 automation hooks
│   ├── utils/toon/      # TOON format tools
│   └── settings files
│
├── AGENTS.md            # This file
└── README.md            # Project overview
```

---

## 4. The `.Codex/` Directory

### What's in it

**40 Skills** - Auto-invoked knowledge domains organized into 10 categories:
- **anthropic/** - AI & Codex (7 skills)
- **aptos/** - Blockchain & Shelby Protocol (17 skills)
- **decibel/** - On-chain trading (1 skill)
- **plaid/** - Banking API (5 skills)
- **shopify/** - E-commerce platform (1 skill)
- **stripe/** - Payments (1 skill)
- **supabase/** - Backend (1 skill)
- **whop/** - Digital products platform (1 skill)
- **expo/** - React Native (4 skills)
- **ios/** - iOS development (1 skill)
- **toon-formatter/** - Token optimization (1 skill)

**Documentation** - NOT bundled in repo. Pull separately using `docpull`:
```bash
pipx install docpull
docpull <docs-url> -o .Codex/skills/<skill>/docs
```
Available documentation (pull as needed):
- Helius: 200+ files - `docpull https://www.helius.dev/docs -o .Codex/skills/helius/docs`
- Stripe: 3,253 files - `docpull https://docs.stripe.com -o .Codex/skills/stripe/docs`
- Supabase: 2,616 files - `docpull https://supabase.com/docs -o .Codex/skills/supabase/docs`
- Expo: 810 files - `docpull https://docs.expo.dev -o .Codex/skills/expo/docs`
- Plaid: 659 files - `docpull https://plaid.com/docs -o .Codex/skills/plaid/docs`

**7 Commands** - Slash commands (5 TOON format + 2 skill marketplace):
- `/analyze-tokens` - Compare JSON vs TOON savings
- `/convert-to-toon` - Full conversion workflow
- `/toon-decode` - TOON → JSON
- `/toon-encode` - JSON → TOON
- `/toon-validate` - Format validation
- `/discover-skills` - Browse SkillsMP marketplace
- `/install-skill` - Install skills from GitHub

**5 Hooks** - Post-tool automation scripts (disabled by default)

**TOON Utils** - Native Zig encoder/decoder (357KB binary, 20x faster)

---

## 5. How Skills Work

Skills **auto-activate** based on conversation context:

| Keyword | Skill | Documentation |
|---------|-------|---------------|
| Stripe API, payments | Stripe | 3,253 docs |
| Whop, memberships | Whop | 212 docs |
| Supabase, PostgreSQL | Supabase | 2,616 docs |
| Aptos, Move language | Aptos | 150+ docs |
| Expo, React Native | Expo | 810 docs |
| Plaid, banking | Plaid | 659 docs |
| Shopify, e-commerce | Shopify | 25 docs |
| TOON, token optimization | TOON Formatter | Tools + guide |

**Skills are hierarchically organized:**
- Shelby Protocol is nested under Aptos (decentralized blob storage on Aptos blockchain)
- Decibel is also Aptos-based but separate as a trading platform
- Plaid has 4 sub-skills (Auth, Transactions, Identity, Accounts)
- Expo has 3 sub-skills (EAS Build, EAS Update, Expo Router)

---

## 6. TOON Format

**What is it?**
TOON (Token-Oriented Object Notation) reduces token consumption by **30-60%** for tabular data.

**When to use:**
- Arrays with 5+ items
- Objects with 60%+ field uniformity
- API responses, logs, metrics, benchmarks

**Example:**
```javascript
// JSON: ~120 tokens
[
  {"method": "GET", "path": "/api/users", "auth": "required"},
  {"method": "POST", "path": "/api/users", "auth": "required"}
]

// TOON: ~70 tokens (40% savings)
[2]{method,path,auth}:
  GET,/api/users,required
  POST,/api/users,required
```

**Complete specification:** `.Codex/utils/toon/toon-guide.md`

---

## 7. Quick Start

### Use skills automatically

Skills activate when relevant keywords are mentioned:

```
User: "How do I create a Stripe subscription?"
Stripe skill activates with 3,253 docs

User: "Build a Whop membership system"
Whop skill activates with 212 docs

User: "This JSON is too big"
TOON formatter activates with optimization tools
```

### Pull documentation (optional)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raintree-technology/claude-starter](https://github.com/raintree-technology/claude-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
