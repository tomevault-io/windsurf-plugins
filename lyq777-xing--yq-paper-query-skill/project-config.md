---
trigger: always_on
description: >-
---


# Paper Query Skill

## What I Do

- Search 7+ academic databases in parallel with automatic retry and fallback
- **Filter: Only show published papers** (journal/conference proceedings, exclude unpublished preprints)
- Deduplicate and merge results by DOI, arXiv ID, or title similarity
- Annotate venue tiers using CCF (A/B/C), CAS-JCR (Q1-Q4), and SCI (Q1-Q4)
- Translate paper titles from English to Chinese
- Generate 2-3 sentence bilingual summaries from abstracts
- Format results as Markdown with direct links to arXiv, DOI, Semantic Scholar
- Auto-save results to `papers/query_<timestamp>.md`

---

## Workflow

### Step 0: Check Configuration

Read `paper-config.json` from the project root. If the file is missing:

> I notice your paper query is not set up yet. Let me help you configure it. Please run `/paper-setup` to create your profile — it takes about 2 minutes.

If the file exists, extract:
- `user.name`, `user.institution`, `user.field_of_study`
- `api_keys` for rate-limited APIs
- `preferences.databases`, `preferences.min_venue_tier`, `preferences.max_results`
- `preferences.sort_by`, `preferences.year_range`, `preferences.output`

**Dynamic Year Range**: If `preferences.year_range` is `null` or not set, default to the last 3 years:
- `year_start = current_year - 3` (e.g., 2023 if today is 2026)
- `year_end = current_year`
- User can specify explicit year range in query to override this default

### Step 1: Parse Query

From the user's query string, extract:
- **Keywords**: core search terms — generate 2-3 query variants (exact phrase, broader terms, synonyms) to maximize recall
- **Authors**: if mentioned (e.g., "papers by Hinton")
- **Year range**: if user specifies (e.g., "--year 2023-2025"), use it. Otherwise, use config default (last 3 years). **CRITICAL: Always apply a year filter** to avoid overwhelming results.
- **Venue filter**: if user mentions a specific conference/journal
- **Topic mapping**: map keywords to CS subfields for venue prioritization

**Important**: The year range defaults to the last 3 years. If the user does not specify a year and the config has `null`, use `current_year - 3` to `current_year`.

### Step 2: Parallel Database Search

Query all configured databases simultaneously using `webfetch`. 
**IMPORTANT**: Request **2x** `preferences.max_results` from each source to compensate for unpublished preprints that will be filtered later.

**General Retry Strategy for all APIs**:
- If 429 (rate limited): wait 5 seconds, retry once. If still fails, try the fallback endpoint.
- If 500/502/503 (server error): wait 2 seconds, retry once. If still fails, skip that source and note it.
- If timeout (>15s): skip and try fallback.
- If 403 (forbidden): skip immediately — this source requires authentication.

**Data Coverage Note**: Semantic Scholar and OpenAlex are meta-search engines that aggregate papers from hundreds of publishers including IEEE Xplore, ACM Digital Library, Springer, Elsevier, and others. Even when IEEE/ACM direct APIs fail, their papers ARE still represented in the results — you just get them through S2/OpenAlex instead of directly. Do NOT treat IEEE/ACM direct API failure as missing data.

#### 2.1 arXiv API (with rate-limiting discipline)

arXiv enforces strict rate limits: **no more than 1 request per 3 seconds**. Violations result in 429 + temporary ban.

**Query strategy** — use a single targeted request:
```
URL: http://export.arxiv.org/api/query?search_query=all:{encoded_keywords}&start=0&max_results={2x_max_results}&sortBy=relevance
```
Parse XML response. Extract: id, title, summary, authors (list), published (date), doi, journal_ref, primary_category, link.

**Rate limit handling**:
- If 429: wait 20 seconds (arXiv ban cooldown), retry once. If still fails, skip — S2 and OpenAlex already cover arXiv papers.
- Do NOT send multiple rapid requests to arXiv. One query per search is sufficient.

**Published-only check**: After parsing, flag papers that have a `journal_ref` or `doi` as PUBLISHED. Papers with neither are preprints — keep them for dedup but mark as unpublished candidate.

#### 2.2 Semantic Scholar API (with rate-limit handling)

**Primary endpoint** — try this first:
```
URL: https://api.semanticscholar.org/graph/v1/paper/search?query={encoded_keywords}&limit={2x_max_results}&year={year_start}-{year_end}&fieldsOfStudy={topic}&fields=title,year,authors,externalIds,url,abstract,venue,citationCount,publicationTypes,openAccessPdf,publicationDate,journal
```
If API key configured, add: `x-api-key: {key}`

**Fallback endpoint** — use if primary returns 429:
```
URL: https://api.semanticscholar.org/graph/v1/paper/search/bulk?query={encoded_keywords}&year={year_start}-{year_end}&fields=title,year,authors,externalIds,url,abstract,venue,citationCount,publicationTypes,journal
```

**No-key workaround** — if both endpoints rate-limit and no API key is configured:
- Inform user: "Semantic Scholar API 请求被限。建议在 https://www.semanticscholar.org/product/api 免费注册 API key 以提高限额。"
- Skip Semantic Scholar, proceed with other databases.

Parse JSON response (`data` array). Extract: title, year, authors, externalIds (DOI, ArXiv), url, abstract, venue, citationCount, publicationTypes, journal.

#### 2.3 DBLP API (with fallback)

**Primary endpoint**:
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lyq777-Xing/yq_paper_query_skill](https://github.com/lyq777-Xing/yq_paper_query_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
