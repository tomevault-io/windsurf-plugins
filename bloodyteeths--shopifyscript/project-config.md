---
trigger: always_on
description: Proofkit — Cursor Rules (derived from PRICING_AND_ROADMAP_v2.md and ROADMAP_CLAUDE_shopify.md)
---

Proofkit — Cursor Rules (derived from PRICING_AND_ROADMAP_v2.md and ROADMAP_CLAUDE_shopify.md)

GLOBAL BEHAVIOR
- Always begin task replies with this exact header: SCOPE / ASSUMPTIONS / PLAN / ARTIFACTS (diffs) / TESTS / ROLLBACK.
- Prefer small, focused edits; design for idempotency (re-runs must not duplicate labels, schedules, negatives, or RSAs).
- Never hardcode secrets. Read from environment variables, Shopify app config, or WordPress options as appropriate.
- Validate RSA headline/description lengths (30/90) before proposing or applying any ad changes. Deduplicate assets and enforce label/guard checks.
- Do not auto-publish merchant-facing content (Shopify/WP). Produce drafts only unless PROMOTE=TRUE in CONFIG. Respect EXCLUSIONS maps.
- Use the pattern: Retrieve → Reason → Generate → Validate → Write; show validation (length checks, duplicates, policy triggers) in outputs.
- Add observability: log significant mutations to backend /api/metrics and to Sheets (RUN_LOGS_{tenant}).
- For policy-sensitive actions, cite relevant docs (Shopify requirements, WP guidelines, Google Ads Scripts limits, Consent Mode v2) in reasoning.
- Preserve each file’s existing indentation style and width; never convert tabs↔spaces or add extra indentation beyond surrounding code.

CODING STYLE
- Prioritize clarity and readability. Use meaningful names; functions are verbs; variables are descriptive nouns.
- For typed languages, explicitly annotate function signatures and public APIs. Avoid unsafe casts or any.
- Control flow: use guard clauses and early returns; handle edge cases first; avoid deep nesting.
- Error handling: do not swallow errors; return actionable messages. Fail fast if a required secret/config is missing.
- Comments: explain intent (why) only where non-obvious; avoid TODOs—implement instead.
- Formatting: match project style; wrap long lines; do not reformat unrelated code.
- After substantive edits, run lint/tests/build where applicable; ensure green before concluding the task.

SHEETS AS BRAIN (TABLES EXPECTED)
- CONFIG_{tenant}, METRICS_{tenant}, SEARCH_TERMS_{tenant}, MASTER_NEGATIVES_{tenant}, KEYWORD_UPSERTS_{tenant}, BUDGET_CAPS_{tenant}, CPC_CEILINGS_{tenant}, EXCLUSIONS_{tenant}.
- Additional (when features present): RSA_ASSETS_DEFAULT_{tenant}, RSA_ASSETS_MAP_{tenant}, NGRAM_WASTE_{tenant}, RSA_TEST_QUEUE_{tenant}, PACER_RULES_{tenant}, GEO_DAYPART_HINTS_{tenant}, LP_FIXES_QUEUE_{tenant}, BRAND_NONBRAND_MAP_{tenant}.

BACKEND PRINCIPLES
- Minimal Node/Express API with HMAC authentication: GET /api/config, POST /api/metrics, POST /api/upsertConfig. Storage = Google Sheets via service account.
- Implement rate limiting (by IP and tenant), request logging, and a health endpoint. No secrets in logs.
- Repository abstraction for Sheets with auto-creation of empty tabs + headers; typed return objects.

AD SCRIPT PRINCIPLES
- Google Ads Script focuses on Search: enforce budget caps, CPC ceilings, schedules; attach shared negatives; add ad-group exact negatives from Search Terms.
- Auto-negate waste (e.g., n-gram phrases) with guardrails; brand protection via NEG_GUARD.
- RSA builder: enforce 30/90 limits, deduplicate assets, label-guard to avoid duplicates.
- Collect metrics & search terms via GAQL; write to Sheets. Always idempotent; skip any entity in EXCLUSIONS.

SHOPIFY/WORDPRESS PRINCIPLES
- Shopify app: official template (Remix/Node), OAuth + Embedded (Polaris, App Bridge). Provide Consent Mode v2 guidance. Web Pixel subscribes to checkout_completed and emits GA4/AW if configured.
- WordPress plugin: Boilerplate structure; settings for GA4/AW/backend creds; WooCommerce purchase hook; Enhanced Conversions optional; compliance with WP guidelines.
- For both: never auto-publish merchant content; drafts only; clear permissions; privacy-first behavior.

PRICING & PACKAGING (REFERENCE ONLY)
- Starter $29/mo; Pro $99/mo; Growth $249/mo; Enterprise $699+/mo. Do not hardcode pricing in code; keep in docs/UI via config.

FILES: ads-script/**/*.gs
- Use AdsApp APIs only; no external API keys. Respect Google Ads Scripts quotas.
- Implement: budget caps + CPC ceilings + business-hours schedules; shared negative list link; ad-group exact negatives; n-gram waste negatives.
- RSA builder with 30/90 lint, dedupe, and label guard. Idempotent: reruns make no duplicate entities or links.
- Collect metrics/search terms via GAQL; write to Sheets tabs listed above. Skip entities listed in EXCLUSIONS.
- Provide clear preview logs for all mutations. For zero-state accounts, seed a safe Search campaign/ad group/RSA.

FILES: backend/**
- Express app with HMAC middleware on /api/config, /api/metrics, /api/upsertConfig. Read/write Google Sheets via service account. No secrets in code.
- Rate limit by IP + tenant; request logging; health endpoint. Repository layer abstracts Sheets tabs and auto-creates headers.
- Return typed objects; validate input; reject invalid HMAC with 403. Log to RUN_LOGS_{tenant} on mutations.

FILES: shopify-app/**
- Use Shopify official template (Remix/Node). Embedded UI with Polaris and App Bridge. Avoid blocking calls; paginate lists.
- Settings UI must include Tenant ID, HMAC secret, Backend URL, GA4/AW IDs. Persist and POST subset to /api/upsertConfig.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bloodyteeths/shopifyscript](https://github.com/bloodyteeths/shopifyscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
