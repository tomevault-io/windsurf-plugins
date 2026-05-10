---
trigger: always_on
description: A background pipeline that builds a psychological + behavioral profile of a user
---

# Second Self — Layer 1 Identity Pipeline

## What this project is
A background pipeline that builds a psychological + behavioral profile of a user
(identity.md) by analyzing their Gmail data and enriching it with Tavily web search.
This profile becomes the "primer" for a digital twin agent that acts as the user.

## What you are building right now
Layer 1 of a 6-layer memory system. Your job is ONLY the identity pipeline:
- Gmail OAuth + full email fetch (all folders)
- Tavily search enrichment
- Analysis passes to extract voice, behavior, relationships, interests
- Writing the final identity.md file to ~/.secondself/identity.md

Do NOT build the twin agent, the VNC layer, the notch UI, or anything else.
Stay scoped to the identity pipeline only.

## Tech stack
- Python 3.11+
- Google Gmail API (google-auth, google-auth-oauthlib, google-api-python-client)
- Tavily Python SDK (tavily-python)
- Anthropic SDK for analysis passes (claude-sonnet-4-20250514)
- python-dotenv for env vars
- All secrets in .env, never hardcoded

## Project structure to build
```
second-self/
├── .env                          # secrets (gitignored)
├── CLAUDE.md                     # this file
├── requirements.txt
├── main.py                       # orchestrates the full pipeline
├── auth/
│   └── gmail_auth.py             # OAuth flow + token management
├── fetch/
│   ├── gmail_fetch.py            # pulls all emails, all folders
│   └── tavily_fetch.py           # web search enrichment
├── clean/
│   └── email_cleaner.py          # strips HTML, signatures, quoted chains
├── analyze/
│   ├── voice_analyzer.py         # style analysis on sent emails
│   ├── topic_extractor.py        # topic/interest extraction across all emails
│   ├── behavior_analyzer.py      # response patterns, habits
│   ├── relationship_mapper.py    # contact scoring → feeds Layer 2.5
│   └── tavily_synthesizer.py     # LLM pass on Tavily results
├── build/
│   └── identity_builder.py       # assembles identity.md from all analysis
└── output/
    └── identity.md               # final output (also written to ~/.secondself/)
```

## .env variables needed
```
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=http://localhost:8080
TAVILY_API_KEY=
ANTHROPIC_API_KEY=
USER_NAME=                # optional — auto-detected from Google sign-in
USER_EMAIL=               # optional — auto-detected from Google sign-in
```

## Gmail OAuth rules
- Scope: https://www.googleapis.com/auth/gmail.readonly (read only, all folders)
- Store token in ~/.secondself/gmail_token.json
- If token exists and is valid, skip OAuth flow — do not re-auth on every run
- If token is expired, refresh silently using the refresh token
- Only trigger the browser consent screen if no valid token exists

## Email fetch rules
- Fetch ALL folders — inbox, sent, spam excluded, trash excluded
- Use labels: fetch INBOX and SENT as separate labeled batches
- Cap: 2000 emails max total, most recent first
- For each email extract: id, threadId, labelIds, subject, from, to, cc,
  date (unix timestamp), body (plain text preferred, html fallback)
- Batch API requests — do not fetch one email at a time
- Store raw fetched emails as a local JSON cache at output/raw_emails.json
  so re-runs don't re-fetch. If cache exists and is less than 24h old, use it.
- Reconstruct threads: group emails by threadId, sort by date ascending

## Email cleaning rules
- Strip all HTML tags, convert to plain text
- Remove quoted reply chains — anything after "On [date] [name] wrote:" or
  lines starting with ">"
- Remove email signatures — detect by looking for "--" separator or
  common signature patterns (phone numbers, titles, company names in last 3 lines)
- Remove forwarded message blocks
- Remove unsubscribe links and footer boilerplate
- If cleaned body is under 10 words, discard the email entirely
- Keep the cleaned emails in memory, do not cache cleaned version

## Tavily fetch rules
- Query 1: user's full name (from USER_NAME env var)
- Query 2: user's name + their email domain (e.g. "Vin @columbia.edu")
- Query 3: user's name + "github" OR "linkedin" OR "twitter"
- Run all 3 queries, deduplicate results by URL
- Store raw Tavily results in output/tavily_raw.json

## Analysis passes — use Claude for all LLM calls
Model: claude-sonnet-4-20250514
Max tokens: 1500 per call
Temperature: 0 (deterministic — this is analysis, not generation)
Always structure prompts to return JSON — parse the response as JSON

### voice_analyzer.py
Input: sent emails only (filter by SENT label)
Analyze:
- avg_sentence_length (count words per sentence, average across corpus)
- vocabulary_markers (top 20 words/phrases used disproportionately vs general English)
- opener_patterns (how they start emails — categorize and count)
- signoff_patterns (how they end emails — categorize and count)
- emoji_frequency (total emojis / total emails, round to 2 decimal)
- question_ratio (sentences ending in "?" / total sentences)
- length_distribution (% short <50 words, % medium 50-200, % long >200)
- tone_descriptor (single word: casual / formal / terse / verbose / warm / direct)

Group sent emails by recipient email domain to detect code-switching:
- internal (same domain as user) vs external vs personal (gmail/yahoo/etc)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [23jmo/second-self](https://github.com/23jmo/second-self) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
