---
trigger: always_on
description: The intelligence layer for an AI-ready WooCommerce store: analytics skills, knowledge resources, prompts, and an AI-readiness scoring engine, registered as WordPress Abilities and exposed through the plugin's own MCP endpoint at `/wp-json/woocommerce-claude/mcp`.
---

# WooCommerce for Claude

The intelligence layer for an AI-ready WooCommerce store: analytics skills, knowledge resources, prompts, and an AI-readiness scoring engine, registered as WordPress Abilities and exposed through the plugin's own MCP endpoint at `/wp-json/woocommerce-claude/mcp`.

WordPress plugin, PHP 7.4+, GPL-3.0-or-later. Public repo — never commit secrets, internal-only URLs, or anything not intended for public distribution.

## At the start of each session

**Read [CONTRIBUTING.md](./CONTRIBUTING.md) first.** It is the authoritative reference for:

- Architecture (`Plugin (PHP) → Abilities API → WC core MCP server`)
- The **privacy rule** — analytics responses are aggregated only; no PII flows to AI by default
- The **merchant-scope rule** — tools never suggest "build a new skill / endpoint"; the merchant can't action that
- WooCommerce tables and known edge cases (refund sub-orders, returning-customer flag, refunds formula)
- Caching (don't use WC core's DataStore cache; use transients with stable cache keys)
- The full how-to for adding a new analytics Skill (questions-first, ability registration, mandatory PHPUnit test, coverage guard)
- Design patterns worth knowing — silence-isn't-signal, guardrail bad/good phrasing pairs, narrative-layer pre-compute, small-N honesty, two-frame responses, mode-switching, tool-vs-resource affordances, custom-header vs OAuth auth, regulatory thresholds

**Re-stating the two highest-stakes rules** because they're load-bearing for every ability description and a single violation ships to a public release:

- **Privacy.** Analytics responses are aggregated counts/sums/averages only. No customer names, emails, or addresses unless the `woocommerce_claude_allow_customer_pii` option is opt-in **and** the response shape clearly justifies it. Default off.
- **Merchant scope.** The AI is talking to a merchant. They can't add a tool, register a REST endpoint, or edit plugin code. Tool descriptions and merchant-facing text MUST NOT suggest "a future X skill would answer this" — substitute with something the merchant can action (a setting, a manual workflow, a connector, or an honest "this isn't something we can answer"). The static guardrail-sweep test (`tests/integration/test-ability-description-guardrails.php`) catches the obvious violations on every `./bin/check`.

## Layout

```
woocommerce-claude/
├── woocommerce-claude.php               # Plugin bootstrap (HPOS declaration, requirements, options migration)
├── includes/
│   ├── class-plugin.php      # Singleton — wires hooks, boots WP MCP adapter, registers own MCP server
│   ├── abilities/            # One file per skill. wc-analytics/* (analytics tools),
│   │                         # woocommerce-claude/* (store/readiness/search tools), wc-knowledge/*
│   │                         # (resources), wc-prompts/* (prompts), woocommerce-claude-integrations/*
│   │                         # (dev-only prototype scaffolds — gated by wp_get_environment_type)
│   ├── api/                  # REST controllers + AnalyticsController (shared SQL helper —
│   │                         # no REST routes; abilities call into it)
│   ├── knowledge/            # Provider pattern (store profile / catalog / product / policy)
│   ├── scoring/              # Engine + 4 factors (product, schema, content, policy)
│   ├── settings/             # WC > Settings > WooCommerce for Claude tab
│   └── telemetry/            # SkillTelemetry + handlers (log, Tracks-gated by opt-in toggle)
├── tests/integration/        # PHPUnit; runs inside wp-env tests-cli container
├── tools/
│   ├── seed-demo-store.php   # 24-month, 5k-order seeded demo store (mt_srand(42))
│   └── mu-plugins/           # dev-only mu-plugins (allow-insecure-transport for HTTP wp-env)
├── skills/                   # Reference Claude Code / Codex skills (catalog-audit,
│                             # product-content-generator, store-health-monitor)
├── bin/
│   ├── check                 # Local CI mirror — PHPCS + composer audit + PHPUnit + DCC
│   └── check-dcc             # Data Consistency Checker (gated; auto-skips if not installed)
├── docs/performance-and-hosting.md
├── .wp-env.json              # wp-env (port 8888, mounts plugin + tools/mu-plugins)
└── .github/workflows/        # ci.yml (PHPCS + PHPUnit) · release.yml (tag → plugin zip)
```

## Local dev

```bash
npx @wordpress/env start          # boots WP 6.9 + WC + this plugin on http://localhost:8888
                                   # afterStart activates WC + WooCommerce for Claude, installs WC pages,
                                   # sets a UK store address (London / GBP)
./bin/check                        # full pre-push gate (PHPCS, composer audit, PHPUnit, DCC)
```

To seed a realistic demo store (deterministic — `mt_srand(42)`):

```bash
npx @wordpress/env run cli -- bash -c "cat > /tmp/seed.php" < tools/seed-demo-store.php
npx @wordpress/env run cli -- wp eval-file /tmp/seed.php
```

## Architecture decisions baked in


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [woocommerce/woocommerce-claude](https://github.com/woocommerce/woocommerce-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
