---
trigger: always_on
description: 이 프로젝트는 AI 에이전트 팀이 전문적인 Word 문서(.docx)를 작성하는 프로젝트다.
---

# AI 책쓰기 프로젝트

## 프로젝트 구조

이 프로젝트는 AI 에이전트 팀이 전문적인 Word 문서(.docx)를 작성하는 프로젝트다.
콘텐츠(제목, 목차, 페르소나, 작성 스타일, 퍼블리싱 사양)는 `user-book-toc.md`에 정의되어 있다. 다른 책을 쓸 때는 그 파일만 교체하면 된다.

| 파일 | 역할 | 교체 대상 |
|------|------|-----------|
| `CLAUDE.md` | 프로세스, 범용 규칙 (이 파일) | 아니오 |
| `user-book-toc.md` | 제목, 페르소나, 목차, 작성 스타일, 퍼블리싱 사양 | **예** (다른 책 쓸 때 이 파일만 교체) |
| `.claude/skills/*.md` | 단계별 실행 스킬 | 아니오 |

## 작성 규칙 (모든 스킬이 준수)

### 범용 규칙 (어떤 책이든 적용)

1. 문서의 모든 섹션은 `user-book-toc.md`의 페르소나로 작성한다
2. 최종 산출물에는 에이전트 내부 역할명이 등장하지 않는다
3. 작성자 표기는 `user-book-toc.md`의 "필명"으로만 한다
4. 전문 용어가 나올 때는 괄호 안에 영문을 병기한다
5. 각 장 끝에 수평선(---)을 넣지 않는다. 장 구분은 제목(Heading)으로만 한다
6. 장 제목, 활용법 제목 형식은 `user-book-toc.md`의 형식을 따른다
7. 독자 지칭은 `user-book-toc.md`의 "독자 지칭"을 따른다
8. 예시의 연도는 `user-book-toc.md`의 "기준 연도"를 따른다 (기능 출시일 등 사실 정보는 실제 연도 유지)

### 스타일 규칙 (책마다 다를 수 있음)

`user-book-toc.md`의 "작성 스타일" 섹션을 따른다.

## 작업 폴더 규칙

### 날짜 기반 프로젝트 폴더

모든 작업 산출물은 **날짜 기반 폴더** 안에 저장한다. 한 권의 책 = 하나의 날짜 폴더.

폴더 명명 규칙:
- 기본: `YYYYMMDD` (예: `20260504`)
- 같은 날 새 프로젝트를 또 시작하면 접미: `YYYYMMDD_01`, `YYYYMMDD_02`, ...
- 날짜는 `bash`의 `date +%Y%m%d`로 얻는다

폴더 구조:
```
draft/
└── 20260504/                  ← 활성 프로젝트 폴더 (이 책 작업 전체가 여기 있음)
    ├── 01_research-notes.md
    ├── 02_outline.md
    ├── 03_draft-v1.md
    ├── 03_draft-v1_part1.md   ← /write 청크 작업 중간물
    ├── 03_draft-v1_part2.md
    ├── ...
    ├── 04_review-red.md
    └── ... (14_review-proofreader.md 까지)

output/
└── 20260504/                  ← 같은 날짜의 출판 산출물
    ├── final.docx
    ├── metadata.md
    ├── verify-report.md
    └── images/
        ├── fig01.png
        ├── fig02.png
        └── ...
```

### 활성 프로젝트 폴더 결정 로직

모든 스킬은 시작할 때 다음 규칙으로 "활성 폴더"를 결정한다:

| 스킬 | 동작 |
|------|------|
| `/research` | 오늘 날짜의 폴더를 **새로 만든다**. 같은 이름이 이미 있으면 `_01`, `_02`로 접미를 붙인다 |
| `/write`, `/review`, `/publish` | `draft/` 하위 폴더 중 **사전순 가장 최신** 폴더를 활성으로 사용한다 |

`/research`가 새 폴더를 만드는 시점에 동일한 이름의 `output/YYYYMMDD/` 폴더도 함께 만든다 (이미지·docx 출력용).

활성 폴더가 결정되면 그 경로를 사용자에게 한 줄로 알리고 작업을 시작한다:
```
활성 프로젝트 폴더: draft/20260504/
```

## 상태 마커 (Stage Completion Marker)

각 산출물 md 파일이 **완성**되었을 때, 파일 마지막에 다음 한 줄을 추가한다:

```
<!-- STAGE_COMPLETE: 01_research-notes -->
```

`STAGE_COMPLETE:` 뒤의 식별자는 파일명에서 `.md`를 뗀 값과 동일하다 (예: `02_outline`, `03_draft-v1`, `08_ensemble-review-1`).

스킬은 다음 단계로 넘어갈지 판단할 때 이 마커의 유무를 본다.
- 마커 있음 → 그 단계는 완료된 것으로 본다
- 마커 없음 → 부분 작성 상태로 보고 같은 파일에 이어쓰기(append)한다

부분 작성 상태에서 절대 다음 단계를 시작하지 않는다.

## 작업 흐름

4개의 핵심 스킬을 순서대로 실행한다. 각 스킬 내부의 세부 단계는 자동으로 진행된다.

각 단계가 완료되면 해당 파일의 상태 마커를 확인한 뒤 다음 단계로 넘어간다. 한 단계가 끝나기 전에 다음 단계를 시작하지 않는다. 대화가 끊기거나 새 세션에서 이어 작업할 때는, 활성 폴더를 자동 감지하고 마커가 없는 파일부터 이어쓰기로 재개한다. 기존 내용을 덮어쓰지 않는다.

### 핵심 워크플로우 (순서대로)

표의 경로에서 `{ACTIVE}`는 위 규칙으로 결정되는 활성 프로젝트 폴더(예: `20260504`)를 말한다.

| 순서 | 스킬 | 내부 단계 | 최종 산출물 |
|------|------|-----------|------------|
| 1 | `/research` | 웹 검색 조사 | `draft/{ACTIVE}/01_research-notes.md` |
| 2 | `/write` | 구조 설계 → 초안 작성 | `draft/{ACTIVE}/02_outline.md`, `draft/{ACTIVE}/03_draft-v1.md` |
| 3 | `/review` | 내부 리뷰(비평, 독자, 합평) → 외부 리뷰(편집자, 마케터, 프루프리더) → 최종 수정 | `draft/{ACTIVE}/04_review-red.md` ~ `draft/{ACTIVE}/14_review-proofreader.md` |
| 4 | `/publish` | 검증 → Word 변환 → 메타데이터 생성 | `output/{ACTIVE}/final.docx`, `output/{ACTIVE}/metadata.md`, `output/{ACTIVE}/verify-report.md` |

표지는 이 하네스에서 다루지 않는다. 필요하면 외부 도구(예: `adobe-design-from-template`)나 디자이너에게 맡긴다.

### 산출물 번호 체계

활성 폴더 안의 모든 md 파일은 생성 순서대로 번호 prefix를 붙인다.

| 번호 | 파일명 | 스킬 | 산출 단계 |
|------|--------|------|-----------|
| 01 | `01_research-notes.md` | `/research` | 웹 조사 |
| 02 | `02_outline.md` | `/write` | 구조 설계 |
| 03 | `03_draft-v1.md` | `/write` | 초안 (Part별 `03_draft-v1_partN.md`로 청크 후 병합) |
| 04 | `04_review-red.md` | `/review` | 내부: 비평가 |
| 05 | `05_draft-v2.md` | `/review` | 내부: 비평 반영 |
| 06 | `06_review-pink.md` | `/review` | 내부: 독자 |
| 07 | `07_draft-v3.md` | `/review` | 내부: 독자 반영 |
| 08 | `08_ensemble-review-1.md` | `/review` | 내부: 합평 1라운드 (5인) |
| 09 | `09_draft-v4.md` | `/review` | 내부: 합평1 반영 |
| 10 | `10_ensemble-review-2.md` | `/review` | 내부: 합평 2라운드 (5인) |
| 11 | `11_draft-final.md` | `/review` | 내부: 합평 통과본 |
| 12 | `12_review-editor.md` | `/review` | 외부: 편집자 |
| 13 | `13_review-marketer.md` | `/review` | 외부: 마케터 (참고용) |
| 14 | `14_review-proofreader.md` | `/review` | 외부: 프루프리더 |

번호는 항상 두 자리(01, 02, ...)이고, 같은 번호의 파일은 덮어쓰기로 갱신한다.

## 품질 기준

- 문서 구조가 `user-book-toc.md`의 목차와 완전히 일치해야 한다
- 문서 분량이 `user-book-toc.md`의 목표 분량을 충족해야 한다
- 페르소나가 처음부터 끝까지 일관되어야 한다
- 내부 참조가 실제 내용과 일치해야 한다
- 기능 정보가 문서 전체에서 일관되어야 한다
- [그림] 플레이스홀더는 matplotlib으로 생성한 실제 이미지로 대체한다
- Word 문서에 `**` 마크다운 기호가 남아 있으면 안 된다
- em dash(—, –)가 본문에 남아 있으면 안 된다
- 어미는 해라체로 통일되어야 한다 (인용 대화·프롬프트 예시 제외)

이 항목 중 자동 검사 가능한 것은 `/publish` 1단계의 `verify.py`로 강제한다.

## 변경 로그 표준

수정 단계(05, 07, 09, 11번 파일)는 본문 첫머리에 다음 형식의 "변경 로그" 섹션을 둔다.

```markdown
## 변경 로그

| 항목 | 심각도 | 출처 | 반영 | 사유 (미반영 시) |
|------|--------|------|------|------------------|
| 5부 활용법 09 도입부 통계 | 🔴 | 04_review-red | ✅ | |
| 어미 통일 (3장) | 🟡 | 14_review-proofreader | ✅ | |
| 6장 위트 보강 | 🟡 | 12_review-editor | ❌ | 페르소나 톤과 충돌 |
| 5부 챕터 순서 재배치 | 🟢 | 12_review-editor | ❌ | 참고 의견, 미반영 |
```

🔴 항목은 모두 ✅로 끝나야 한다. 🔴이 ❌로 남으면 그 단계는 통과시키지 않는다.

## 백업 정책

다음 트리거에서만 자동 백업한다 (모든 백업은 `{ACTIVE}/_backup/`에 ISO 타임스탬프로 보관).

| 트리거 | 백업 대상 | 백업 파일명 |
|--------|-----------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airoasting/book_publishing](https://github.com/airoasting/book_publishing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
