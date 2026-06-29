---
trigger: always_on
description: Guide on how lists we're made and how it works.
---

# AGENTS

Guide on how lists we're made and how it works.

## Refs

REF1: [README](./README.md)

## Rules

- Always read latest :REF1 before any action
- Always run `npx prettier -c README.md --write` after any modification on :REF1
- Always fetch actual name, URL, pricing and other fields value of entry before adding, updating or modifying :REF1
- Always use **monthly** plan pricing as reference, never use temporarily discounts, yearly discounts as permanent pricing
- URL last uneless `/` should be striped. No unnecessary `/` on end
- Never assume, always prefer to fetch fresh metadata and check. Always verify before finalize

## Columns

- Name: Provider name with URL provided
  - Do a small typo fixes, if necessary. Example: `[qodi](https://qoddi.com)` -> `[Qoddi](https://qoddi.com)`
- Minimal plan: minimum paid plan name with pricing
- Trial: is provider gives free trial
  - Like how DigitalOcean gives 60-day trial or 14-day trials for other providers
  - Example values: `1-Month` if 1-month free, `$50` credit if provider gives credit on sign-up
  - Do not include `free`, `trial` suffixes to help compact table
  - Only include consumer/self-serve sign-up credits. Do not include enterprise, startup, application-only, sales-assisted, or high-value non-consumer credits
- Free: Free plan name (if exists)
- Open Source: Is provider gives discount or free plan to OSS/FOSS projects

## Entries

- Provider must be checked with `grep` and case-insensitive
- New entry must not exist in :REF1
- All list entries separated by categories (aka topics), see list of [categories](./README.md#contents)
- Some of list entries should contain **status** to mark project as one of [status](./README.md#status)
- If entry **Free plan** exists, the **Minimum plan** should not show $0 or any zero value
- As **Minimum plan** should be shown one step higher than **Free plan** price (if free plan exists) or **Lowest paid plan**
- Allowed currencies are: USD ($) and EUR (€) with their corresponding sign.
  - Any other values should be converted to USD ($) at value of published date.
  - For sorting, compare prices by normalized USD value, not by the printed number.
  - When a row uses EUR (€), fetch the current EUR-to-USD exchange rate and use the converted USD value only for ordering.
  - If two rows show the same numeric price but different currencies, do not treat them as equal; for example, `5 \$/m` sorts before `€ 5/m` when EUR is worth more than USD.
  - Example with USD: (5 \$/m)
  - Example with EUR: (€ 5/m)
- Entries are sorted by normalized **Price** then by **Name**
- Before finalizing an entry change, check adjacent rows in the affected category for currency-conversion ordering errors.

## Per-section providers

### LLM/Inference API

- Model price is for `gpt-oss-120b` where available
- Token pricing reference models should be released after 01 August 2025
- If `gpt-oss-120b` is unavailable, use the closest `gpt-oss` model or leave pricing link without forced model price
- Before changing the token reference model or calling a model universal, extract every provider from `README.md` `LLM/Inference API` -> `Tokens` and verify the model provider-by-provider.
- Universal model verification must include all current Tokens providers by name, with evidence/status for each provider; do not skip entries such as `together.ai`, providers with reference-style links, or rows without explicit pricing.
- If any Tokens provider cannot be verified for the exact model, do not call the model universal; document the missing provider(s) and keep the fallback rule.
- Model price schema: `[Pricing](https://groq.com/pricing) (INPUT/OUTPUT \$/1M token)`
- Model price example: `[Pricing](https://groq.com/pricing) (0.15/0.60 \$/1M token)`

## Examples

### Adding new VPS provider

Name: Megahost
URL: <https://megahost.kz/vps>
Minimum plan: SSD-mini (3000 ₸ converted into USD; 6.5 \$/m)
Trial: not exists
Free plan: not exists
Open Source: - (empty)

### Default example

```diff
 | [AlfaHost](https://alfahost.io)               | [Pricing](https://alfahost.io/?tab=vps) (6 \$/m)                            | No                 | No                    |                  |
+| [Megahost](https://megahost.kz/vps)           | SSD-mini (6.5 \$/m)                                                         | No                 | No                    |                  |
 | [Adaptable](https://adaptable.io)             | Hobby (7 \$/m)                                                              | No                 | Public-repos only     |                  |
 | [rdp.monster][rdp-monster-ref]                | [Basic](https://rdp.monster) (9 €/m)                                        | No                 | No                    |                  |
```

### Lower pricing new VPS provider

```diff
+| [000anon](https://000anon.io)                 | [Pricing](https://000anon.io/?tab=vps) (5 \$/m)                             | No                 | No                    |                  |
 | [AlfaHost](https://alfahost.io)               | [Pricing](https://alfahost.io/?tab=vps) (6 \$/m)                            | No                 | No                    |                  |
```

### Higher pricing new VPS provider

```diff

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dalisoft/awesome-hosting](https://github.com/dalisoft/awesome-hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
