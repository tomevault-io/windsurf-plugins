---
trigger: always_on
description: 디자이너가 AI와 일관되게 일하기 위한 4단계 파이프라인.
---

# design-flow-harness

디자이너가 AI와 일관되게 일하기 위한 4단계 파이프라인.
실행 환경: **Claude Code 전용** (`.claude/agents` · `.claude/skills`).
대상: 모바일 앱 (iOS/Android, 390×844).

레퍼런스 수집부터 Figma 화면 생성까지, 매 단계 게이트를 통과하며 진행한다.
자연어로도, 슬래시 명령으로도 동작한다.

---

## 6가지 원칙

### 1. 레퍼런스 없이 시작하지 않는다

Phase 1의 analysis.md 없이는 Phase 2로 갈 수 없다.
근거 없는 화면 구조는 만들지 않는다.

### 2. 규칙이 확정되지 않으면 Figma에 손대지 않는다

design/03-design-rules/design-rules.md 상단 `status: confirmed` 없이는
figma-builder가 절대 실행되지 않는다. 이 파일이 규칙 SSOT다.

규칙 안의 토큰은 primitive(값) → semantic(의도) 2계층으로 적는다.
색상·간격·radius·size 가 대상이며, 타이포는 role 기반 텍스트 스타일을 쓴다.
화면에 들어갈 이미지는 §I 표에 "어느 슬롯에 라이브러리의 어느 파일"로 적는다.
이미지는 생성하지 않는다 — `design/assets/characters/` 에 있는 파일만 쓴다.
아이콘도 그리지 않는다 — lucide 이름을 적고 CDN 에서 받는다.

### 3. 각 단계 게이트를 통과해야 다음으로

4개 Phase 사이에 게이트 4개. 각 게이트는 3중 확인:

- 스크립트 자동 검증
- 에이전트 자체 판단
- 사용자 승인

하나라도 실패하면 다음 Phase 진입 금지.

### 4. 기본값이 항상 있다

사용자가 "모르겠어요"라고 해도 진행이 멈추지 않는다.
scripts/default-tokens.md의 기본값을 적용하고 산출물에 가정 로그로 남긴다.

### 5. 산출물이 곧 상태다

각 Phase의 진행 상황은 design/ 폴더가 자체적으로 알려준다.
세션이 끊겨도 폴더만 있으면 다음부터 이어갈 수 있다.
"어디까지 했지?"를 물어볼 필요가 없다.

### 6. 자연어로도, 명령어로도 동작한다

"레퍼런스 뽑아줘" (자연어) = `/collect-references` (슬래시)
사용자 편의에 따라 선택. 결과는 동일한 에이전트가 실행.

---

## 표준 워크플로

```
Phase 1 · 레퍼런스     → 게이트 1 →
Phase 2 · 화면 구조    → 게이트 2 →
Phase 3 · 디자인 규칙  → 게이트 3 →
Phase 4 · Figma 생성   → 게이트 4 → 🎉 완료
           (tokens → components → screens)
```

각 게이트 통과 없이는 다음 Phase 진입 불가.

---

## 에이전트 라우팅

| 요청 유형     | 자연어 예시                                  | 에이전트               | 슬래시 명령         |
| ------------- | -------------------------------------------- | ---------------------- | ------------------- |
| 레퍼런스 수집 | "레퍼런스 뽑아줘", "경쟁사 분석"             | reference-collector    | /collect-references |
| 레퍼런스 분석 | "분석해줘", "패턴 뽑아줘"                    | reference-analyzer     | /analyze-references |
| 화면 구조     | "화면 구조 짜줘", "화면 목록"                | structure-builder      | /build-structure    |
| 디자인 규칙   | "규칙 만들어줘", "디자인 시스템"             | design-rules-generator | /generate-rules     |
| Figma 생성    | "Figma 화면 만들어줘"                        | figma-builder          | /create-figma       |
| 최종 검증     | "검증해줘", "audit"                          | design-auditor         | /audit-design       |
| 스냅샷 추출   | (자연어 없음 · 코디네이터가 백그라운드 기동) | snapshot-runner        | (없음)              |

snapshot-runner 는 사용자가 직접 부르는 에이전트가 아니다. figma-builder 가 build-log 에
`snapshot: requested` 를 남기면 코디네이터가 띄운다.

---

## 산출물 구조

```
design/
├── 01-references/         ← Phase 1
│   ├── raw/               (스크린샷 3개+)
│   └── analysis.md        (분석 + 패턴 통합)
│
├── 02-structure/          ← Phase 2
│   ├── screens.md         (화면 5개+ 정의)
│   └── flows.md           (시나리오 2-3개)
│
├── 03-design-rules/       ← Phase 3 (SSOT)
│   ├── design-rules.md    ⭐ 유일한 규칙 SSOT
│   ├── tokens.md
│   ├── components.md
│   ├── design-direction.md / direction-options.html (방향 비교)
│   └── preview.html
│
├── 04-screens/            ← Phase 4
│   ├── figma-file-key.txt      (사용자가 만든 Figma 파일 키)
│   ├── figma-snapshot.json     (audit 입력 · figma-snapshot.js 로만 추출)
│   ├── build-log.md
│   ├── build-manifest.json    (캡처 상태·입력/산출물 해시)
│   ├── visual-review.json     (상태별 시각 검수 근거)
│   ├── audit-report.md
│   ├── audit-structural.json   (figma-audit.mjs 출력)
│   ├── fix-list.md             (audit FAIL 시만 생성)
│   └── screenshots/            (이미지까지 채워진 완성본)
│
└── assets/                ← 이미지 라이브러리 (사람이 채운다 · 에이전트 생성 금지)
    └── characters/        (§I 표의 `파일` 열이 가리키는 곳 · check-assets.mjs 가 대조)
```

---

## 절대 원칙

- **design-rules.md `status: confirmed` 없이 figma-builder 실행 금지**
- **토큰은 primitive → semantic 2계층. semantic 에 값 직결 금지**
- **컴포넌트·화면은 semantic 토큰만 바인딩** (primitive 직접 사용 금지)
- **Phase 순서 건너뛰기 금지** (1 → 2 → 3 → 4 순차)
- **게이트 실패 시 다음 Phase 진입 금지**
- **design/03-design-rules/design-rules.md 는 유일한 규칙 SSOT**
- **Figma 파일은 사용자가 만든 것만 사용한다** (에이전트가 새 파일 생성 금지)
- **snapshot 은 scripts/figma-snapshot.js 로만 추출한다** (추출 코드 즉흥 작성 금지 · 경량화는 스크립트의 `__PROFILE__=docs` 만)
- **snapshot 추출은 snapshot-runner 가 백그라운드로 한다.** figma-builder 는 lint 0건이면 요청만 남기고 다음 STAGE 로 간다. audit 전에는 세 페이지 스냅샷 전부 PASS 필수
- **토큰 문서 프레임은 scripts/figma-token-docs.js 로만 그린다** (규격은 docs/token-docs-spec.md · 즉흥 작성 금지)
- **컴포넌트 문서 페이지(`02b Component Docs`)는 scripts/figma-component-docs.js 로만 그린다** (규격은 docs/component-docs-spec.md · 원본 세트는 `02 Components` 최상위에 두고 문서에는 인스턴스만 · 스냅샷 대상 아님)
- **일반 컨테이너는 내용을 감싼다 (세로 HUG).** 고정 높이는 선언된 컴포넌트와 실제 클리핑·스크롤이 설정된 viewport만 허용 (check-layout.mjs 검사)
- **화면 이미지는 design-rules.md §I 표가 가리키는 `design/assets/characters/` 파일만 쓴다** (이미지 생성·외부 URL 금지)
- **아이콘은 lucide 이름으로 적고 CDN 에서 받는다** (손으로 그리지 않는다 · 버전 고정)
- **이미지 슬롯을 빈 채로 두고 Phase 4 를 끝내지 않는다** (게이트 4에서 FAIL)
- **각 에이전트는 자기 담당 폴더 외 편집 금지**
- **사용자 승인 없이 다음 Phase로 자동 진행 금지**

---

## UI 품질과 검수 증거

- 구조 담당은 `screen-contract.json`에 필수 상태와 주 행동 정책(single/collection/none)을 작성한다.
- 규칙 담당은 대표 화면 2안·선택 이유를 `design-direction.md`에 기록한다. 기존 승인 방향은 재사용한다.
- 최종 HTML 시안에는 실제 콘텐츠와 라이브러리 이미지를 넣고 필수 상태까지 비교한다.
- 텍스트 폭·자연스러운 줄바꿈·이미지의 내용 식별·탭 선택 상태·스크롤을 검수한다.
- 주 행동은 실제 탭 대상의 `harnessAction` 메타데이터로 검사한다. 이름이나 색만으로 판정하지 않는다.
- 개발 중에는 기존 비동기 runner를 유지한다. 최종 검수 때는 Figma 수정을 동결하고
  `capture:begin` → 동일 ID의 세 페이지 추출·새 PNG → `capture:seal` → audit → 시각 검수 순서로 진행한다.
- snapshot 공유 파일의 병합은 직렬화한다. 수정이 있으면 캡처를 다시 시작한다.
- `visual-review.json`에 화면별 점수와 관찰 근거를 남긴다. 각 차원 4/5 이상,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [figmatutor-info/designflowharness](https://github.com/figmatutor-info/designflowharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
