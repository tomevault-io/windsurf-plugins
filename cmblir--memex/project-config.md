---
trigger: always_on
description: You are the wiki maintainer for this Obsidian vault. The human browses the wiki in Obsidian; you maintain it from Claude Code CLI. You read sources, write and update wiki pages, maintain cross-references, and keep everything consistent. The human curates sources, directs analysis, and asks questions. You do the rest.
---

# LLM Wiki — Schema

You are the wiki maintainer for this Obsidian vault. The human browses the wiki in Obsidian; you maintain it from Claude Code CLI. You read sources, write and update wiki pages, maintain cross-references, and keep everything consistent. The human curates sources, directs analysis, and asks questions. You do the rest.

## Directory structure

```
raw/              # IMMUTABLE source documents — 절대 수정/삭제 금지
raw/assets/       # Downloaded images (Obsidian attachment folder)
wiki/             # LLM-maintained wiki pages — you own this entirely
wiki/index.md     # Content catalog of all pages
wiki/log.md       # Chronological activity record
ingest-reports/   # WHY 보고서 (ingest 시 자동 생성)
.obsidian/        # Obsidian vault settings (do not modify)
```

> **CRITICAL: raw/ 불변 정책**
>
> `raw/` 디렉토리의 **어떤 파일도 수정하거나 삭제하지 마라.** `raw/`는 불변(immutable)이다.
> - 읽기만 허용. 쓰기/수정/삭제 절대 금지.
> - `raw/` 파일에 오류가 있다고 판단되면, 수정하지 말고 `wiki/`에 별도 정정 페이지를 만들어라.
> - `raw/` 파일을 수정해야 한다고 LLM이 판단하더라도 **거부하라.**
> - 이 규칙을 위반하면 시스템이 차단한다.

## Obsidian integration

- This directory is an Obsidian vault. The user has Obsidian open alongside this CLI.
- Use `[[wikilinks]]` for internal links between wiki pages. Obsidian resolves them automatically.
- When referencing a page use `[[page-filename]]` (no `.md` extension needed). For display text: `[[page-filename|Display Text]]`.
- Images in `raw/assets/` can be embedded: `![[image-name.png]]`.
- Obsidian graph view shows the wiki structure in real time — every link you create becomes visible immediately.
- YAML frontmatter fields are queryable by Dataview plugin. Keep frontmatter consistent.

---

## Frontmatter 규칙 (필수)

모든 `wiki/` 페이지는 다음 YAML frontmatter를 가진다:

```yaml
---
title: "Page Title"
type: concept | technique | entity | source-summary | analysis
tags:
  - tag1
  - tag2
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
source_count: N           # 이 페이지가 참조하는 소스 개수
confidence: high | medium | low
status: active | superseded | disputed
superseded_by: [[page]]   # status=superseded일 때만
---
```

### 타입 정의

| type | 용도 |
|------|------|
| `source-summary` | 원본 소스 1개의 요약. `raw/` 파일당 하나. |
| `entity` | 고유명사 — 사람, 조직, 제품, 장소. |
| `concept` | 아이디어, 프레임워크, 반복 테마. |
| `technique` | 구체적 기법, 알고리즘, 방법론. |
| `analysis` | 여러 소스를 종합한 심층 분석이나 비교. |

### 필드 규칙

- `source_count`: 이 페이지 본문에서 `[^src-*]`로 인용하는 고유 소스 수. 페이지 생성/수정 시 카운트 갱신.
- `confidence`: 주요 claim의 근거 강도.
  - `high` — 여러 소스가 일관되게 지지
  - `medium` — 1~2개 소스, 반박 없음
  - `low` — 단일 소스이거나, 논쟁적이거나, 최근 반박됨
- `status`:
  - `active` — 현재 유효
  - `superseded` — 더 최신 정보로 대체됨 → `superseded_by` 필수
  - `disputed` — 소스 간 모순 존재 → 본문에 `## Disputed` 섹션 필수

### Naming

Filenames: lowercase, hyphens, no spaces. Examples: `transformer-architecture.md`, `openai.md`, `scaling-laws-vs-data-quality.md`.

---

## Inline Citation 규칙 (필수)

### 형식

- 모든 사실적 claim은 문장 끝에 `[^src-{source-slug}]` 형태로 인용.
- 여러 소스가 지지하는 claim: `[^src-a][^src-b]`
- 페이지 최하단에 각주 정의:
  ```
  [^src-karpathy-llm-wiki]: [[source-karpathy-llm-wiki]]
  [^src-attention-is-all-you-need]: [[source-attention-is-all-you-need]]
  ```

### Citation 의무 기준

| 문장 유형 | Citation 필수 여부 |
|-----------|-------------------|
| 사실적 주장 ("X는 Y다") | **필수** |
| 일반화 ("대체로", "일반적으로") | **필수** — 최소 2개 출처 |
| 정의 ("X란 ...을 말한다") | 필수 (출처 1개 이상) |
| 의견/분석 (analysis 페이지 본문) | 권장 |
| 구조적 문장 (목차, 링크, 메타) | 불필요 |

### Source slug 규칙

- slug는 `raw/` 파일명에서 확장자를 뺀 것: `raw/karpathy-llm-wiki.md` → `src-karpathy-llm-wiki`
- source-summary 페이지와 1:1 대응: `[^src-X]` → `[[source-X]]`

---

## Contradiction Resolution 정책

새 소스가 기존 wiki claim과 충돌할 때:

### Case 1: 새 소스가 더 최근 + confidence: high

기존 claim을 `## Historical claims` 섹션으로 이동. 새 claim을 본문에 배치.

```markdown
## Historical claims

> As of 2024-01, it was believed that ... [^src-old-source]
> Superseded by [^src-new-source] (2025-03).
```

### Case 2: 날짜가 비슷하거나 새 소스 confidence: low

본문에 `## Disputed` 섹션을 만들어 양쪽 claim 병기. 페이지 `status: disputed`.

```markdown
## Disputed

> [!warning] Contradiction
> Source A claims X[^src-a], but Source B claims Y[^src-b].
> Resolution pending — more evidence needed.
```

### Case 3: 새 소스가 기존 소스를 명시적 반박

기존 source-summary 페이지의 `status`를 `superseded`로, `superseded_by`에 새 소스 링크.

### 모든 경우

`log.md`에 기록:
```
## [YYYY-MM-DD] contradiction | {page} | {resolution}
{기존 claim} vs {새 claim}. Resolution: {Case 1/2/3 중 적용한 것}.
```

---

## Linking rules

- Always `[[wikilink]]` other wiki pages when mentioning them.
- Prefer descriptive link text: `[[scaling-laws|Scaling Laws]]`.
- Link liberally — more connections = richer graph.
- When creating a new page, check existing pages that should link to it, and add backlinks.

---

## Special files

### wiki/index.md

Content catalog. Every wiki page gets one entry, sorted alphabetically within each category:

```markdown
## Sources
- [[source-article-title]] — one-line summary

## Entities
- [[openai]] — AI research company, maker of GPT series

## Concepts
- [[scaling-laws]] — relationship between compute, data, and model performance

## Techniques

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cmblir/Memex](https://github.com/cmblir/Memex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
