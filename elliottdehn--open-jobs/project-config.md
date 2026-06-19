---
trigger: always_on
description: You are an agent helping a person find a job. This file tells you exactly how to use the
---

# AGENTS.md — How to find a job with this dataset

You are an agent helping a person find a job. This file tells you exactly how to use the
**open-jobs** dataset to do that well. Read it fully before you start.

The dataset is a single Parquet file at **https://download.jobscream.com/open-jobs.parquet**,
refreshed in place daily. Grab it with `python3 download.py` (resumable), or read it straight from
the URL with `stream.py`. It is a deduplicated snapshot of every currently-open role pulled from
16 applicant tracking systems (Greenhouse, Lever, Ashby, Workday, SmartRecruiters, Workable,
Breezy, Personio, Paylocity, Dayforce, Recruitee, Pinpoint, Recruiterbox, JobScore, Crelate,
Eightfold). Roughly a million rows. Each row carries metadata, the full job description as
Markdown, precomputed embeddings, and ~34 structured fields extracted from the JD.

It is a snapshot, not a history. Yesterday's file is gone. Work with what is here now.

## How you should work

Write **small Python scripts** to filter and rank, and produce a **self-contained HTML file** the
person opens in their browser to explore the results. No database, no server, no special tooling.
Everything here runs with:

```
pip install pandas pyarrow numpy openai
```

The canonical pipeline is **hull -> learn -> rank** (section 4): draw the convex hull of eligible
roles, gather the LLM's pairwise judgments inside it, aggregate those into a ranking, then write an
HTML report (section 5). Keep the scripts readable so the person can rerun and tweak them.

---

## 1. Load it

```python
import pandas as pd, numpy as np
# either download first (python3 download.py) then read the local file,
# or read straight from the URL:
df = pd.read_parquet("https://download.jobscream.com/open-jobs.parquet")
df = df[df["jd_markdown"].str.len() > 0]      # drop the few rows with no description
print(len(df), "jobs")
```

The two embedding columns come back as arrays of 1536 floats. Do NOT stack all million into one
matrix (that is ~6 GB). You filter first, then build the embedding matrix only on the survivors
(section 4).

---

## 2. Schema

**Identity & apply**
`id, ats, company, url` — `url` is where the person applies. Always surface it.

**Content**
`title, jd_markdown` — the full description, cleaned to Markdown.

**Where & when**
`location, posted_at, remote` (bool), plus structured geography below.

**Embeddings** (OpenAI `text-embedding-3-small`, 1536-dim, float32) — see section 4.
- `title_embedding`, `jd_embedding`: one vector each (the posted title; the JD). JD similarity is the
  strongest signal for "what is this role."
- `alt_titles_embedding`: a **list of vectors**, one per entry in `alt_titles` (aligned 1:1), NOT a
  single vector. Each alt title was embedded separately so a query matches the closest *variant*, not
  a blurred average. Match a title query by taking the **max** cosine over a row's alt vectors.

**Structured facets** (extracted by an LLM from each JD; use for exact filtering)

- Role: `level` (Intern…C-Level/Unknown), `function` (engineering, data, design, product, sales,
  marketing, ops, security, finance, hr, legal, research, support, healthcare, education,
  skilled-trade, other), `sub_function`, `role_summary`, `employment_type`,
  `years_experience_min`, `years_experience_max`, `education_min`, `management` (bool),
  `key_responsibilities` (list)
- Titles: `alt_titles` (list, MOST specific first then broadening — your lexical recall handle)
- Location & eligibility: `is_remote` (bool), `work_mode` (fully_remote/remote_first/hybrid/
  onsite/unknown), `city`, `region`, `country_code` (ISO alpha-2), `country_required` (bool),
  `remote_scope` (e.g. us-only, us-canada, emea, latam, apac, global), `relocation`,
  `timezone_requirement`, `travel_required`
- Comp & legal: `salary_min_k`, `salary_max_k` (thousands), `salary_currency` (ISO 4217),
  `equity` (bool), `visa_sponsorship` (yes/no/unknown), `security_clearance` (bool)
- Company: `company_does`, `industry`, `company_stage`
- Skills & tags: `skills` (required/core), `nice_to_have`, `tags`

**Unknown sentinels.** Do not mistake these for real values:
`""` (string), `"unknown"` (enum), `-1` (number), `[]` (list), `false` (boolean).
Critically: `salary_min_k = -1` means "not stated," not "free." Filter accordingly.

---

## 3. First, build the candidate profile

Before querying, extract these from the person's resume and what they tell you. Ask if unclear.

- target `function` and `level` (and acceptable adjacent levels)
- core `skills` they have, and which they want to use
- location: which `country_code`(s) they can legally work in, and whether they need remote
- if remote: which `remote_scope` works for them (a `us-only` role is useless to an EU resident)
- comp floor (in a currency), and whether `equity` matters
- `visa_sponsorship` need (true/false)
- deal-breakers: `employment_type`, `security_clearance`, `travel_required`, relocation

Write this down explicitly. Every step below references it.

---

## 4. The search: hull -> learn -> rank

Three small tools ship alongside this file and run as a pipeline. The philosophy: use cheap exact
filters to draw the smallest set that still CONTAINS every relevant role (the "convex hull"), then

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elliottdehn/open-jobs](https://github.com/elliottdehn/open-jobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
