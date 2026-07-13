---
trigger: always_on
description: This repository is a Memex research vault. It is meant to be used by agents through the `mmx` CLI, not by editing the SQLite database directly.
---

# AGENTS.md

This repository is a Memex research vault. It is meant to be used by agents through the `mmx` CLI, not by editing the SQLite database directly.

## Identity

You are working inside Oh My AI Company.

Your job is to read, write, and maintain structured research assets:

- companies and products
- people and founders
- investors and investments
- source evidence
- touchpoints for monitoring
- traffic snapshots
- notes, concepts, and methods
- images and other research assets

Facts must come from captured sources. Do not invent facts. If a page cannot be read or only returns navigation/cookie/empty content, mark it as failed or partial and do not use it as evidence.

Boundary: this vault stores company research assets, not social media operations assets. Social/community content can be saved as `source.item` only when it is evidence for a company/product/person/investor research question. Do not create `social.account`, `social.post`, or `social.analytics.snapshot` objects in this vault.

## Required Tooling

Memex product repo:

```text
https://github.com/yan5xu/memex
```

Use the `mmx` CLI from Memex. If you built Memex locally, either put `mmx` on `PATH` or set `MMX` to your local binary path.

```bash
MMX=mmx
```

Use the repository root as the vault path:

```bash
cd /path/to/oh-my-ai-company
VAULT=$(pwd)
```

Always run commands with `-C "$VAULT"` unless your current working directory is definitely the vault root.

## First Commands

Run these before doing meaningful work:

```bash
MMX=mmx
VAULT=$(pwd)

$MMX --help
$MMX -C "$VAULT" vault info
$MMX -C "$VAULT" status
$MMX -C "$VAULT" issues
```

Expected health check:

```text
issues count = 0
```

If `issues` is nonzero, inspect and fix the issue before writing new research unless the user explicitly asks otherwise.

## Storage Model

Memex stores:

- `.memex/memex.db` for types, fields, objects, and links.
- `bodies/*.md` for long-form Markdown body content.
- `assets/` for screenshots, product images, diagrams, and source media.
- `memex.graph-views.json` for Graph v2 views.

Do not edit `.memex/memex.db` manually.

Use `mmx` commands for object fields and links. Markdown bodies can be edited directly, but after direct edits run:

```bash
$MMX -C "$VAULT" body refresh <object-id>
```

## Read Workflow

Check schema:

```bash
$MMX -C "$VAULT" type list
$MMX -C "$VAULT" field list company
$MMX -C "$VAULT" field list source.item
$MMX -C "$VAULT" field list note
```

Find objects:

```bash
$MMX -C "$VAULT" query company --where 'title=Browserless' --select id,title,website,tags
$MMX -C "$VAULT" query source.item --where 'about_company=company.browserless' --select id,title,platform,evidence_level,quality
```

Read object:

```bash
$MMX -C "$VAULT" get company.browserless --body-preview 800
$MMX -C "$VAULT" links company.browserless
$MMX -C "$VAULT" backlinks company.browserless
```

## Write Workflow

Prefer `upsert` for objects and `source add` for evidence. Use `--body-stdin` for substantial body content.

Create or update company:

```bash
cat <<'EOF' | $MMX -C "$VAULT" upsert company company.example \
  name="Example" \
  title="Example" \
  status=active \
  website="https://example.com" \
  tags=ai-agent,developer-tool \
  --body-stdin
# Example

Write the company dossier here. Keep facts and judgments separate.
EOF
```

Add source evidence:

```bash
cat <<'EOF' | $MMX -C "$VAULT" source add source.example-home \
  --title "Example homepage" \
  --url "https://example.com" \
  platform=Website \
  item_type=profile \
  evidence_level=S1 \
  quality=full \
  about_company=company.example \
  --body-stdin
Summarize the source, capture key facts, and state evidence boundaries.
EOF
```

Add a human judgment note:

```bash
cat <<'EOF' | $MMX -C "$VAULT" upsert note note.example-takeaway \
  title="Example product takeaway" \
  kind=takeaway \
  author=researcher \
  about_company=company.example \
  created_at=2026-07-13 \
  --body-stdin
Write analysis, takeaway, question, decision, or reflection here.
EOF
```

Create strong links:

```bash
$MMX -C "$VAULT" link company.example founders person.example-founder
$MMX -C "$VAULT" link source.example-home about_company company.example
```

Markdown body links use `[[object.id]]`. After writing Markdown links:

```bash
$MMX -C "$VAULT" body refresh <object-id>
```

## Core Types

High-frequency types:

- `company`: company/product subject.
- `person`: founder, investor, operator, observer.
- `investor`: venture firm, angel, accelerator, strategic investor.
- `investment`: investor-company funding relation.
- `source.item`: one source/evidence item.
- `touchpoint`: durable monitoring entry such as website, X account, GitHub repo, Product Hunt page, LinkedIn page, search query.
- `traffic.snapshot`: Similarweb/Semrush/Google Trends/GA/GSC traffic record.
- `note`: human judgment, takeaway, question, decision, reflection.
- `concept`: reusable concept or market/product/GTM pattern.
- `method`: research or tooling method.

Do not use `touchpoint` for one-off articles. One-off articles belong in `source.item`.

## Evidence Rules

Evidence levels:

- `S1`: official or primary source.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yan5xu/oh-my-ai-company](https://github.com/yan5xu/oh-my-ai-company) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
