---
trigger: always_on
description: > `CLAUDE.md` § 도메인 프레임워크의 상세 정본. LLM은 페이지를 쓰거나 갱신하거나 위키에 질문할 때 이 규약을 따른다. 규약을 바꾸면 여기서 바꾸고 `_meta/changelog.md`에 기록한다.
---

# 위키 페이지·인덱스·라우팅 규약 (정본)

> `CLAUDE.md` § 도메인 프레임워크의 상세 정본. LLM은 페이지를 쓰거나 갱신하거나 위키에 질문할 때 이 규약을 따른다. 규약을 바꾸면 여기서 바꾸고 `_meta/changelog.md`에 기록한다.

---

## 0. 핵심 모델 — 라우터·샤딩·정본화 (먼저 읽기)

LLM Wiki의 검색은 "전부 읽기"가 아니라 **라우팅**이다. 위키가 수천 페이지로 커져도 **질문 1개당 읽는 토큰이 거의 일정**하게 유지되는 게 목표다. 이를 위한 4개 기둥:

1. **index = 라우터(MOC)**, 카탈로그가 아니다. `index.md`는 "모든 페이지 목록"이 아니라 **"질문 의도 → 어느 타입/샤드를 펼칠지"** 정하는 얇은 진입점이다. 엔티티 줄은 index에 두지 않는다.
2. **타입별 하위 인덱스 + 샤딩.** 실제 엔티티 카탈로그는 `indexes/{type}.md`에 둔다. 한 타입이 커지면 **정본명 첫 글자로 ≤50K 토큰 샤드**로 쪼갠다 (§8).
3. **정본화(aliases).** 표기 흔들림(Parasite/기생충, 샬라메/Timothée Chalamet)을 **정본명 1개**로 모은다. 정본명이 곧 **샤드 라우팅 키**다 (§4).
4. **2단 라우팅 (Phase A / Phase B).** 질문 처리는 "어디를 열지 정하기(Route, 샤드 안 읽음)"와 "지정 샤드·페이지 읽기(Search)"를 분리한다 (§9). 라우팅이 샤드를 읽지 않아야 샤딩의 토큰 절감이 지켜진다.

> 규모가 작을 땐 이 구조가 과해 보이지만, **메커니즘을 처음부터 박아둬야** 데이터가 쌓일 때 자연스럽게 라우터→타입 인덱스→샤드로 성장한다. 소규모에선 한 단계가 다음 단계를 겸한다 (§9 성장 경로).

---

## 1. 페이지 타입

| 타입 | frontmatter `type` | 위치 | 1:1 대상 |
|------|--------------------|------|----------|
| 소스 요약 | `source` | `30-wiki/{topic}/sources/` | raw 소스 1개당 1페이지 |
| 엔티티 | `entity` | `30-wiki/{topic}/entities/` | 인물·조직·장소·제품·작품 |
| 개념 | `concept` | `30-wiki/{topic}/concepts/` | 이론·방법론·용어 |
| 타입 인덱스 | `index` | `30-wiki/{topic}/indexes/{type}.md` | 타입 1개당 1개(+샤드) |
| 주제 라우터 | `index` | `30-wiki/{topic}/index.md` | 주제 1개당 1개 |
| 루트 라우터 | `index` | `30-wiki/index.md` | 위키 전체 1개 |
| 종합 개요 | `overview` | `30-wiki/{topic}/overview.md` | 주제 1개당 1개(선택) |
| 정본 사전 | `aliases` | `30-wiki/{topic}/aliases.md` | 주제 1개당 1개 |
| query 파일백 | `query` | `50-queries/` | 가치 있는 질의 결과 |

**도메인 세분(선택):** `entity`가 많아지면 도메인에 맞게 하위 타입으로 나눌 수 있다 (예: 영화 도메인 → `people`/`works`/`series`). 이때 폴더·인덱스도 타입별로 분리한다. 기본은 `entity` 하나로 시작하고, 한 종류가 수십 개를 넘으면 분리를 검토한다.

**신뢰 등급(tier):** 모든 엔티티/개념 페이지는 `tier`를 단다 — `reviewed`(원본 추출·사람 확인) 또는 `auto`(웹/추론 lazy 생성, 미검수). `auto`는 `auto-generated.md` 대장에 등록한다 (§10).

---

## 2. frontmatter 스펙 (타입별)

### source
```yaml
type: source
title: "소스 원제목"
source_file: 20-raw/2026-06-14-article.md   # 처리완료 원본 역참조 (compile이 raw로 이동 후 경로)
topic: "주제 슬러그"
summary: "1~2문장 + 검색 키워드 5~10개"   # 이 문장이 sources/index 줄로 재사용됨
ingested: 2026-06-14
author: ""        # 있으면
url: ""           # 있으면
tags: []
provenance: extracted
```

### entity / concept
```yaml
type: entity            # 또는 concept
canonical: "정본명"      # 라우팅 키 (첫 글자가 샤드 결정)
aka: []                 # 같은 대상의 다른 표기 (aliases.md에도 등재)
topic: "주제 슬러그"
summary: "1~2문장 + 키워드 5~10개"   # ★ 이 문장이 indexes/{type}.md 의 줄 description으로 그대로 재사용됨
tags: []
sources: []             # 이 페이지를 뒷받침하는 raw/소스 id들
tier: reviewed          # reviewed | auto
provenance: extracted   # extracted | inferred | ambiguous | web-enriched
status: active          # active | stub | deprecated
updated: 2026-06-14
```

`summary`는 **인덱스의 원천**이다 — 잘 쓰면 인덱스가 자동으로 좋아진다. 1~2문장 정의 + 검색 키워드를 반드시 포함한다.

---

## 3. 페이지 본문 골격 (고정 섹션)

ingest/compile 때 "어디에 쓸지"를 결정적으로 만들기 위해 타입별 섹션을 고정한다. 빈 섹션은 `_(아직 없음)_`으로 남겨 grep 가능하게 둔다.

- **source**: `**TL;DR:**` → `## Key claims` → `## Entities & concepts` → `## How this updated the wiki` → `## Notable quotes`
- **entity/concept**: `**정의:**`(BLUF) → `## 요약` → `## Key facts` → `## 관계` → `## Open questions / 모순` → `## Sources`

템플릿 실물은 `40-templates/{source,entity,concept}.md`.

---

## 4. 네이밍 & 정본화 (aliases.md)

- 파일명 = **kebab-case 슬러그**, 안정적(한 번 정하면 안 바꿈 — 링크 깨짐 방지). 예: `napoleon-bonaparte.md`
- 한글 엔티티는 한글 슬러그 허용(공백→하이픈). 예: `기동전.md`
- 소스 슬러그는 날짜 prefix 권장: `2026-06-14-article-title.md`

### 정본명 규칙 (라우팅 키)
- 엔티티마다 **정본명 1개**를 정하고 frontmatter `canonical`에 둔다. **정본명 첫 글자가 샤드를 결정**한다 (§8).
- 같은 대상의 다른 표기는 **새 파일을 만들지 말고** frontmatter `aka` + 중앙 `aliases.md`에 등재한다.
- **정본화 = 검색 입구.** 질문이 "Parasite"로 와도 `aliases.md`에서 `기생충`으로 바꾼 뒤 라우팅한다.

### aliases.md (주제별 정본 사전)
`30-wiki/{topic}/aliases.md`에 `별칭/표기 → 정본명` 매핑을 누적한다.
```markdown
| 표기/별칭 | 정본명 | 타입 |
|-----------|--------|------|
| Parasite, 기생충, 寄生虫 | 기생충 | entity(work) |
| 샬라메, Chalamet | Timothée Chalamet | entity(person) |
```
compile이 새 엔티티를 만들 때 별칭이 보이면 여기에 추가한다. query는 라우팅 전에 이 표를 먼저 읽는다.

---

## 5. 링크 & 교차참조

- 위키 내부 참조는 `[[상대경로/슬러그]]`. 예: `[[entities/napoleon-bonaparte]]`, `[[sources/2026-06-14-article]]`
- 모든 **사실 주장**은 뒤에 provenance 링크: `나폴레옹은 코르시카 출신이다 [[sources/2026-06-14-article]]`
- 출처 없는 추론·합성은 `(추론)`/`확인 필요`로 표시.
- **링크는 대상 페이지가 실제 있을 때만** 건다. 없는 엔티티는 plain text로 두고(씨앗), 질문 시 lazy 승격한다 (§10) — 죽은 링크를 양산하지 않는다.

### 동명 disambiguation (경로 명시 링크)
같은 이름이 여러 타입/폴더에 있으면(예: `entities/Dune`이 작품 페이지와 시리즈 페이지 양쪽) **경로 명시 링크**로 가린다:
```markdown
[[works/Dune|Dune]]  (단일 작품)  vs  [[series/Dune|Dune]]  (프랜차이즈)
```
그리고 **라우터(index)의 「동명 충돌」 노트에 그 이름을 등재**해, 라우팅 시 양쪽 샤드를 모두 펼치게 한다.

---

## 6. 모순 처리

새 소스가 기존 주장과 충돌하면 **덮어쓰지 말고** 양쪽을 보존하고 명시:
```markdown
## Open questions / 모순
> ⚠️ Contradiction: [[sources/A]]는 X라 하고 [[sources/B]]는 Y라 한다. 미해결.
```
`/lint`가 `grep -rn "⚠️ Contradiction" 30-wiki/`로 전수 추적한다. (동명 disambiguation은 모순이 아니라 §5의 경로 링크로 처리 — 구분할 것.)

---

## 7. index.md = 라우터 (MOC, 카탈로그 아님)

**핵심: index는 "읽는 목록"이 아니라 "어디로 갈지 정하는 라우터"다.** 엔티티 줄은 index에 두지 않는다 — 그건 `indexes/{type}.md`의 일.

### 루트 라우터 (`30-wiki/index.md`)
- frontmatter `tags: [index, moc, router]`.
- 주제 목록 + 주제별 페이지 수 + 전역 허브(overview·aliases) 링크만.
- "질문이 어느 주제인지" → 해당 주제 라우터로 보낸다.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fivetaku/llm-wiki](https://github.com/fivetaku/llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
