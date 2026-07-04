---
trigger: always_on
description: Schema and harness document for the CMDS LLM Wiki vault. Defines the 3-layer architecture (Raw Sources / Wiki / Schema), ingest-query-lint operations, file conventions, and frontmatter standards. This is the single source of truth for LLM behavior in this vault.
---


# AGENTS.md — LLM Wiki Schema

This file is the **Schema Layer** of the CMDS LLM Wiki. It governs how LLMs (Codex, Cursor, etc.) read, write, and maintain this vault.

> **Architecture**: Karpathy LLM Wiki Pattern
> - Raw Sources = 소스코드 (immutable)
> - Wiki = 실행 파일 (LLM이 관리)
> - Schema = 이 문서 (AGENTS.md)

---

## ⚠️ CRITICAL RULES

### Indentation Rules
- **YAML frontmatter**: 2 SPACES (절대 탭 금지)
- **Markdown body**: TAB (절대 스페이스 금지)

### Wikilink Rules
- YAML 내 wikilinks는 반드시 큰따옴표: `"[[link]]"`
- Markdown body에서는 따옴표 없이: `[[link]]`

### Mermaid Rules
- **모든 노드/엣지 라벨은 큰따옴표**로 감쌀 것 — 한글·특수문자 안정성
	- ✅ `A["시작"] --> B{"선택?"}`
	- ❌ `A[시작] --> B{선택?}`
- **`[/` 로 시작하는 라벨 금지** — trapezoid 도형 기호로 파싱됨 (lexical error)
	- ❌ `C[/query 스킬]`
	- ✅ `C["/query 스킬"]` 또는 `C["query 스킬"]`
- **엣지 라벨도 따옴표 권장**: `B -->|"한글 라벨"| C`
- 라벨 안에 마크다운(`**bold**`, `[[wikilink]]`) 금지 — 렌더 깨짐

### Pre-Flight Checklist (Before Every Write/Edit)
- [ ] YAML frontmatter uses 2 SPACES
- [ ] Markdown body uses TAB
- [ ] Wikilinks in YAML are quoted: `"[[link]]"`
- [ ] Mermaid node/edge labels are quoted: `A["label"]`
- [ ] Arrays use proper format: `- value`
- [ ] Dates use ISO 8601: `YYYY-MM-DD`
- [ ] `description` field present and in English
- [ ] File saved in correct layer folder

---

## Essential (Post-Compact)

> 컨텍스트 압축 후에도 반드시 기억:
> 1. **YAML: 2 SPACES** / **Body: TAB**
> 2. **Wikilinks in YAML: 큰따옴표** `"[[link]]"`
> 3. **Mermaid 라벨: 큰따옴표** `A["label"]` / `[/` 로 시작 금지
> 4. **3 Layers**: Raw Sources (immutable) → Wiki (LLM-maintained) → Schema (this file)
> 5. **Operations**: Capture Tabs → Inbox → Ingest → Query → Verify/Audit → Lint (+Status/Reindex/Refresh)
> 6. **필수 프로퍼티 7개**: type, aliases, description (English), author, date created, date modified, tags
> 7. **Core Context 먼저 읽기**: 모든 operation 전에 [[Core Context]] 로 사용자 목적·철학 정렬
> 8. **미래의 나에게 보내는 편지**: `/ingest` 는 반드시 수집 목적 1회 질문 → `collectionPurpose` 프로퍼티에 기록

---

## 🧭 Core Context (반드시 먼저 로드)

**모든 capture / inbox / ingest / query / lint 전에 [[Core Context]] 를 먼저 읽는다.**

해당 노트는 메인 볼트 `{your-mothership-vault-name}` 의 9 system files (precedence 1~9, 2026-05-22 기준 DESIGN.md 추가) + 핵심 에세이 5편에서 추출된 사용자 맥락 snapshot 이다. {your-name}의 정체성·철학·7 재활용 축·CMDS 9 categories 를 담고 있으며, 이 맥락 없이는 LLM Wiki 의 모든 operation 이 "목적 없는 자동 정리" 로 전락한다. 9 system files alias 표는 아래 표 및 [[Core Context]] §8 참조.

### (옵션) 메인 볼트 9 시스템 파일 — 원 저자 CMDSPACE 구성 예시 (precedence 순)

Mothership 볼트가 없는 standalone 사용자는 이 표를 건너뛰어도 됩니다. 원 저자 기준: 6개 공개 + 3개 비공개 (vendor·product 전용).

| # | Alias | 경로 | 역할 | 공개 |
|:-:|-------|------|------|:----:|
| 1 | `@CMDS-CLAUDE` | `{your-mothership-vault-name}/CLAUDE.md` | HOW — Claude Code 기술 규칙 | 공개 |
| 2 | `@CMDS-AGENTS` | `{your-mothership-vault-name}/AGENTS.md` | HOW — 타 AI 에이전트 규칙 (Codex, Cursor, Windsurf) | 공개 |
| 3 | `@CMDS-Antigravity` | `{your-mothership-vault-name}/ANTIGRAVITY.md` | HOW — Google Gemini / Antigravity IDE 전용 | 비공개 |
| 4 | `@CMDS-Context` | `{your-mothership-vault-name}/CMDS.md` | WHY/WHAT — 시스템 철학·사용자 프로필 | 공개 |
| 5 | `@CMDS-Guide` | `{your-mothership-vault-name}/🏛 CMDS Guide.md` | STANDARDS — 7 프로퍼티·템플릿·camelCase | 공개 |
| 6 | `@CMDS-HQ` | `{your-mothership-vault-name}/🏛 CMDS Head Quarter.md` | WHERE — 87 서브카테고리 네비게이션 | 공개 |
| 7 | `@CMDS-Brain` | `{your-mothership-vault-name}/BRAIN.md` | PERSONA — {your-name} brain profile (Gobi 앱 entry) | 비공개 |
| 8 | `@CMDS-BrainPrompt` | `{your-mothership-vault-name}/BRAIN_PROMPT.md` | PERSONA — Agent Rules of Engagement | 비공개 |
| 9 | `@CMDS-DESIGN` | `{your-mothership-vault-name}/DESIGN.md` | VISUAL — v4.3 design constants · Anti-Slop · skill ↔ surface mapping | 공개 |

최신본 읽기: `Read("{PATH_TO_YOUR_MOTHERSHIP_VAULT}/{file}")` 또는 `mcp__qmd__query` (user-scope, cwd 무관).

[[Core Context]] 은 `snapshot_date` 기준. 30일 이상 오래되면 lint 가 flag → re-snapshot.

---

## Vault Overview

이 볼트는 **Karpathy LLM Wiki Pattern**을 구현한 LLM 전용 지식 베이스입니다.

- **목적**: LLM이 raw sources를 컴파일하여 persistent, structured wiki를 유지
- **철학**: RAG(매번 검색+합성)가 아닌, 한 번 컴파일된 위키가 compounding artifact로 성장
- **연결**: CMDSPACE 메인 볼트(`{your-mothership-vault-name}`)의 satellite vault

### 메인 볼트 연결

| 항목 | 값 |
|------|-----|
| 메인 볼트 경로 | `{PATH_TO_YOUR_MOTHERSHIP_VAULT}` |
| 이 볼트 경로 | `{PATH_TO_YOUR_LLM_WIKI}` |
| Cross-reference | `source-vault` 프로퍼티로 메인 볼트 노트 참조 |

---

## 3-Layer Architecture

### Layer 1: Raw Sources (`10. Raw Sources/`)

**불변층** — 원본 자료를 그대로 보관. 절대 수정하지 않음.

```
10. Raw Sources/
├── 11. Articles/     # 웹 기사, 블로그 포스트
├── 12. Papers/       # 학술 논문, 기술 보고서
├── 13. Books/        # 도서 노트, 챕터 요약
├── 14. Transcripts/  # 강연, 팟캐스트, 영상 전사
├── 15. Clippings/    # 웹 클리핑, 스크랩
└── 16. AI Research/  # ChatGPT/Gemini/Grok/Claude/Perplexity 선행 조사 묶음
```

**규칙**:
- 원본 텍스트를 그대로 보존
- 수정이 필요하면 Wiki 페이지에서 재해석
- `type: raw-source` frontmatter 사용
- `date ingested` 프로퍼티로 인제스트 시점 기록

### Layer 2: The Wiki (`20. Wiki/`)

**LLM 관리층** — LLM이 직접 작성하고 업데이트하는 지식 페이지.

```
20. Wiki/
├── 21. Concepts/     # 추상 개념 (Attention, Transformer, RLHF, ...)
├── 22. Entities/     # 사람, 조직, 제품 (OpenAI, Karpathy, GPT-4, ...)
├── 23. Guides/       # How-to, 튜토리얼, 실전 가이드

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnfkoo951/cmds-llm-wiki](https://github.com/johnfkoo951/cmds-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
