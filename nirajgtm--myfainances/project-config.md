---
trigger: always_on
description: This repo is **public**. Personal data lives in a separate
---

# Repo guardrails for Claude (and human contributors)

This repo is **public**. Personal data lives in a separate
`finance-data/` directory outside the repo. The rules below exist so
nothing personal accidentally lands in committed code, prompts, docs,
or quirks files.

If you are about to commit anything, **run** `bash scripts/check-pii.sh`
first. It exits non-zero on any pattern that looks personal.

## What "personal" means here

Anything tied to a specific user or their financial life. In particular:

1. **Names of real people.** Account holders, family members.
2. **Account numbers** (full or last4), tag/transponder IDs, plan IDs,
   participant IDs, policy numbers, envelope/mailing IDs.
3. **Cities, addresses, ZIP codes, counties.** Especially the user's
   home or service address. Generic "USA" or `<city>` is fine.
4. **Real merchant names** the user has actually transacted with
   (e.g., "Silver Spoon Diner Sunnyvale CA"). Distinct from generic
   chain matchers used for categorization defaults — see "Kept by
   design" below.
5. **Specific dollar amounts** that look like real transactions, real
   employer-match lump sums, real APRs from a single statement, real
   tier prices from a single utility bill.
6. **Specific co-brand product names** the user holds (e.g., "Old Navy
   Encore Mastercard", "bp rewards Visa", "Capital One Venture X"). The
   issuer family name (Capital One, Barclays, FNBO) is fine — the
   specific product name is not.
7. **Employer mentions.** Plan name, plan sponsor, plan ID.
8. **Real timestamps** in schema example values that look like real
   ingest activity (`"ingested_at": "2026-05-04T14:22:01Z"`). Use
   placeholders like `"<ISO timestamp UTC>"`.
9. **Email addresses, phone numbers** that aren't the issuer's
   public customer service line.

## Where personal data tends to leak

- `skill/institutions/<slug>.md` "Quirks observed" sections: the LLM
  appends bullets it learned during ingest. Bullets MUST describe a
  structural pattern, not the specific transaction that exposed it.
  Bad: `Robinhood concatenates merchant + city like 'DECCANMORSELSSUNNYVALECA'`.
  Good: `Robinhood concatenates merchant + city + 2-letter state with no delimiter (MERCHANTNAMECITYSS); split by trailing 2-letter state code`.
- Example values in schema docs (`skill/core/db-schema.md`,
  `skill/extraction/*.md`, `skill/features/*.md`). Use placeholders:
  `<merchant>`, `<account_id>`, `<TICKER>`, `<Employer Name>`.
- LLM prompt strings in `scripts/parser.py`, `scripts/ingest.py`,
  `scripts/research_card_benefits.py`. Examples for the model must
  be generic enough that they could fit any user.
- `examples/*.example.yaml` — these get copied into every fresh
  install. Use placeholders only.

## Redaction patterns (apply in this order)

1. Name → `<account-holder>` or just delete.
2. Last4 / account number → `<last4>` or `XXXX-XXXX`.
3. Specific city → drop the geography entirely if not load-bearing,
   else `<city>`.
4. Real merchant transaction example → `<merchant>` or
   `MERCHANTNAMECITYSS` for pattern-shape examples.
5. Specific dollar amount → drop unless it's a generic default
   (e.g., $500 large-txn threshold).
6. Specific co-brand product → issuer family name only.
7. Real ISO timestamp → `<ISO timestamp UTC>`.

## Kept by design (NOT personal)

These look brand-specific but are functional defaults that help every
user:

- `skill/categorization/default-rules.yaml` — categorization regex
  rules (`(?i)\bWHOLE FOODS\b`, `(?i)APPLE\s*MUSIC`, `(?i)NETFLIX`,
  etc.). These are product matchers; they apply to anyone with that
  subscription/merchant.
- `web/frontend/src/shared/lib/aliases.ts` — search alias dictionary
  mapping common merchant names to category keywords.
- `web/frontend/src/shared/lib/perkIcon.tsx` — keyword matcher that
  picks an icon for a perk based on its name.
- Institution slugs and quirks files that name only the issuer
  (`capital_one`, `chase`, `fidelity`, …) — the integration needs to
  recognize statements from these issuers.

## Process for new contributions

When adding a feature that involves:

- **A new institution.** The first ingest auto-creates
  `skill/institutions/<slug>.md`. Before committing, read the
  generated quirks and confirm every bullet is generic — no specific
  account numbers, transactions, or employer mentions. Genericize
  with `<placeholder>` style.
- **A new schema field.** Pick example values that are clearly
  illustrative (`<merchant>`, `12345`, dummy ISO date). Avoid real
  ticker symbols, real plan IDs, real merchant names.
- **A new LLM prompt.** Examples shown to the model must be generic.
  The parser already instructs the LLM to "describe the structural
  pattern, not the specific account or merchant names that exposed
  it" — keep that line.

## Verifying before commit

```sh
bash scripts/check-pii.sh                # all tracked files
bash scripts/check-pii.sh --staged       # only what's staged for the next commit
```

Returns 0 + green checkmark when clean. Returns 1 + a list of matched
files when anything looks personal. Run it any time you've touched
prompt strings, schema examples, or institution MDs.

The scanner's built-in patterns are **purely structural** — they match

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nirajgtm/myfAInances](https://github.com/nirajgtm/myfAInances) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
