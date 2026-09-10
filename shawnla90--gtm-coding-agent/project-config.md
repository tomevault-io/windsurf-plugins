---
trigger: always_on
description: You are inside the Moltsets reachability starter. It takes a contact list, asks Apollo whether each person is still at the company, asks Moltsets to grade the email address A through F, and renders a color-coded Google Sheet where every row keeps a channel. A bad grade is a routing decision, not a dead contact.
---

# Moltsets Reachability Starter

You are inside the Moltsets reachability starter. It takes a contact list, asks Apollo whether each person is still at the company, asks Moltsets to grade the email address A through F, and renders a color-coded Google Sheet where every row keeps a channel. A bad grade is a routing decision, not a dead contact.

## On "help me set up"

Walk through these checks one at a time:

1. **Moltsets key**: Check for `.env` in this directory. If missing, `cp .env.example .env` and paste the key from app.moltsets.com. `.env` is gitignored and must never be committed. If the user keeps keys in a local secrets vault (a SQLite db outside any repo), do not ask them to paste: `export SECRETS_DB=~/.gtm-vault/vault.db` and the scripts read `secrets(key, value)` from there. See "The secrets-vault explainer" below.

2. **Apollo key (optional)**: Same file or vault, `APOLLO_API_KEY`. It powers `--employment apollo|both`, a second employment source for comparison. The pipeline runs without it.

3. **Google Sheets auth**: Check for `~/.config/gspread/token.json`. If missing, `python3 setup_oauth.py` and walk them through the consent flow. They need a Google Cloud project with the Sheets and Drive APIs enabled.

4. **Dependencies**: `pip install -r requirements.txt`

5. **Contacts**: `sample_contacts.csv` (25 fictional rows on `.example` domains) or their own CSV. The loader accepts the short schema (`first_name,last_name,title,company,domain,email,linkedin_url`) or a raw Apollo people export with its original headers.

6. **Budget first**: `python3 budget.py`. Three free calls. Shows the two record pools, the phone-token balance, and the endpoint catalogue. Do this before any batch.

7. **Run it**: `bash run.sh` (first pass only) or `bash run.sh my_list.csv --full` (employment check + second pass).

## The pipeline

```
init_db.py -> grade.py -> score.py -> build_sheet.py
```

- `init_db.py` loads the CSV into SQLite at `data/reachability.db`. Idempotent. `--country`, `--limit`, `--exclude <file>` filter the load.
- `grade.py` is the waterfall. Per contact: `reverse_linkedin_lookup` on the LinkedIn URL (still there, or moved? plus the graded address when present; `--employment apollo|both` swaps in or adds Apollo `people/match`) -> Moltsets `reverse_email_lookup` when still ungraded -> on 404 or F, optional `search_business_profile_by_name` by name + company DOMAIN (accept only same-domain A/B; `--second-pass-endpoint people` keeps `search_people`, which went 0 for 181) -> LinkedIn-only rows go to `linkedin_to_best_email` -> optional capped `linkedin_to_mobile_phone` for dead-email rows. If the row carries a verifier verdict, a delta class is written next to the grade. Every call is logged; every row is committed as it finishes.
- `score.py` computes title relevance x grade multiplier and ranks the top 3 sendable per company with persona diversity.
- `build_sheet.py` renders up to 11 tabs via the vendored `lib/sheet_engine.py` (Disagreements and Verifier vs Moltsets appear when a verifier verdict exists). Emails obfuscated by default; `--redact-names` for public stills; `--summary-only --share anyone_reader` for a linkable sheet with no rows.
- `import_graded.py` loads rows graded elsewhere (header aliases: grade, molt_email, still_at_company, zb_status, pool, tier, route, delta_class) into the same database, recomputing routing and deltas where missing, so the sheet can be built without an API call.
- `REACHABILITY_DB=data/other.db` in front of any script keeps a second list in its own database and URL file.
- `budget.py` explains the four units that meter Moltsets, using free calls only.

## What Moltsets grades mean (from developer.moltsets.com)

| Grade | Meaning | Action |
|---|---|---|
| A | valid: known reply, open, or click | send |
| B | known send, no bounce | send, smaller batches |
| C | catch-all domain | your call: small monitored segment |
| D | hard invalid: bounce, complaint, or spam trap | never send |
| F | no data | re-verify, or treat like C |
| 404 | not in the graph | a coverage gap, not a verdict; costs nothing |

The docs' rule: suppress D at the list level; segment C and F away from A/B rather than dropping them. Note the common mix-up: F is "no data", D is the hard invalid. Do not swap them.

## What the route column means

| Route | Who lands here | What to do |
|---|---|---|
| `email` | A/B on the company's domain | sequence them |
| `email_low_volume` | C catch-all | small segment, watch bounces |
| `linkedin` | not found after the second pass, or no email | connection note first |
| `linkedin_then_phone` | grade D | never email; LinkedIn, then a mobile if tokens allow |
| `resource` | Apollo says they moved | find them at the new company; the old email is stale |
| `second_pass` | 404 or F and the second pass has not run yet | run `grade.py --second-pass` |
| `hold` | free-mail address, or A/B on a different domain | a human looks |

## Cost model (four units, none interchangeable)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shawnla90/gtm-coding-agent](https://github.com/shawnla90/gtm-coding-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
