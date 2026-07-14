---
trigger: always_on
description: Plannode UI 아이덴티티·컴포넌트 패턴·반응형 표준 (Svelte 셸 + 파일럿 시각 일관성)
---


# Plannode UI 아이덴티티 표준

SvelteKit 메인 셸(`+page.svelte` 등)과 파일럿이 그리는 노드·패널이 **같은 브랜드 톤**을 쓰도록 하는 기준이다. 새 화면·모달·배지를 추가할 때 아래를 우선 맞춘다.

**구현 단일 소스:** UI 토큰은 가능하면 `:root` CSS 변수(`--pn-brand`, `--pn-brand-ink` 등)와 본 문서를 함께 맞춘다. 레거시 하드코드 `#6b4ef6` 계열은 **`#631EED`(로고 Primary)** 로 치환하는 방향으로 통일한다.

## 1. 브랜드·색 (Color) — 로고 SVG 기준

공식 로고 팔레트: 배지 **`#631EED`**, 워드마크 **`#2C155A`**. 상호작용·포커스·CTA는 **Primary = `#631EED`** 로 통일한다.

| 용도 | 참고 값 | 비고 |
|------|-----------|------|
| **Primary (브랜드 보라)** | `#631EED` | 로고 배지색 · CTA·포커스·노드 선택·미니맵 뷰포트·프로젝트 카드 현재 강조 |
| **Primary 진하게 / 호버** | `#5519D4`, `#4a1499` | 그라데이션·호버·드롭다운 글자 강조 |
| **Brand ink (워드마크 진보라)** | `#2C155A` | 강조 텍스트·보조 CTA 글자·태그·스플래시 강조 (`#5b21b6` 대체) |
| **연한 브랜드 배경** | `#ede9fe`, `#f3f1ff`, `#f0ecff` | 아이콘 배경·칩·테두리 없는 보조 버튼 |
| **배경(웜 라이트)** | `#f5f5f0` | `#R`, html/body, AI 뷰 배경 |
| **서페이스/카드** | `#fff`, `#faf9f7` | 모달·입력·노드 카드 |
| **구분선·보더** | `#e0dbd4`, `#e8e4de`, `#ece8e2` | 카드·툴바·표 |
| **뮤트 텍스트** | `#aaa`, `#888`, `#666` | 보조 라벨·메타 |
| **시맨틱** | TDD `#fff1f0`/빨강, AI `#f0fdf4`/초록, CRUD `#eff6ff`, 경고 `#fffbeb` | PRD 패널 인라인·배지와 정렬 |

파일럿 전역 클래스(`:global(.nd)`, `:global(.cp0)` …)는 **위 Primary·Brand ink** 와 같은 계열로 유지한다.

## 2. 타이포·폰트

- **본문 UI:** 한글 **`Pretendard`** (웹폰트), 라틴·숫자 보조 **`Noto Sans KR`** — `app.html`에서 로드 후 `#root`에 스택 적용 (`'Pretendard', 'Noto Sans KR', …`).
- **시스템 폴백:** `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`.
- **코드·트리 덤프:** `monospace` (PRD 기능 트리 블록 등).
- **크기 룰 오브 썸:** 라벨 11px, 본문 12–13px, 모달 제목 15px(`h3`), PRD 헤더 20px, 로고 13px + 서브 10px (`by pseries`).

## 3. 레이아웃·레이어

- **루트:** `#root` = `100vw` × `100vh`; 앱 컬럼 `#R` = 세로 flex, 툴바는 `absolute` 상단 `#TB` + 본문 `#VIEWS`.
- **툴바 `#TB`:** 반투명 화이트 `rgba(255,255,255,0.74)`, **그림자 없음**, `z-index: 50`. 바 자체 `pointer-events: none`, 자식만 `auto`.
- **모달:** `.mbg` 풀스크린 + 어두운 오버레이, `.mo` 카드 `border-radius: 16px`, `z-index: 6000`대.
- **토스트:** `#TST` 하단 중앙.
- **계정 시트:** `z-index: 8000`대.

## 4. 프로젝트 관리 모달 (표준 UI 구조)

진입: `$showProjectModal` · 공통 메뉴 `+` / 「새 프로젝트」 등. **동일 모달**에 **신규 생성 폼** + **이 기기 프로젝트 목록** + (해당 시) **초대/가져오기**가 나온다. **폼·버튼 시각 규격의 기준 구현**은 `src/routes/+page.svelte`(프로젝트 모달 블록·`<style>`)이다.

### 4.1 영역 맵 (클래스)

| 영역 | 클래스·역할 |
|------|----------------|
| 오버레이 | `.mbg` — 반투명 보라 틴트 배경, 클릭 시 닫기(self) |
| 카드 쉘 | `.mo` — 배경 `#fff`, **`border-radius: 16px`**, 패딩 기본 14px, 그림자·최대 높이 스크롤 |
| 프로젝트용 와이드 쉘 | `.mo.mo-wide.pm-scroll.pm-proj-shell` — 세로 flex, **헤더 고정·본문만 스크롤**, 상하 패딩 **21px**·좌우 **14px** |
| 헤더 | `.pm-proj-head` — 제목 `h3`「프로젝트 관리」(15px·진한 `#1a1a1a`), 닫기 `.mcl`(연라벤더 배경 `#f0ece8`, radius 6px) |
| 본문 스크롤 | `.pm-proj-body` — `overflow-y: auto`, 모바일 구간에서 스크롤바 숨김 옵션 |
| 신규 폼 컬럼 | `.proj-form-col` — 아래 **4.2** |
| 목록 컬럼 | `.pl` — `#PLT` 섹션 타이틀, `#PLC` 카드 리스트 |
| 프로젝트 행 | `.prow` → `.pcard` (현재 `.pcard-acp`, 공유 `.pcard-shared`) |
| 행 내부 | `.pdl`, `.pc`, `.pi`, `.pif`, `.pn2`, `.pm2`, `.ct`, `.pacl` 등 기존 계약 유지 |
| 초대 블록 | `.inv-panel`, `.inv-row`, `.inv-load`, `.inv-hint` |
| 모바일 스크롤 힌트 | `.pm-scroll-hint-wrap` |

### 4.2 폼 컬럼 `.proj-form-col` (프로젝트 생성·날짜·설명·노드맵 옵션·배지·가져오기)

| 항목 | 규격 |
|------|------|
| 배치 | 세로 `flex`, **`gap: 15px`** (필드 간 세로 리듬 기준값) |
| 상단 구역과 목록 구역 | `.pl` 은 위쪽에 **`margin-top: 16px`**, **`padding-top: 14px`** 로 폼과 시각 분리(같은 모달 카드 `#fff` 안) |

### 4.3 공통 텍스트 필드 `.fi` (input / textarea)

모달·다른 화면에서 **한 줄·여러 줄 입력**의 기본 스킨으로 통일한다.

| 속성 | 값 |
|------|-----|
| 배경 | `#faf9f7`(웜 서페이스 — 카드 `#fff`와 구분) |
| 테두리 | **없음**(플랫 필드). 레거시 파일럿 `index.html` 등 **보더형 `.fi`** 와 병행되는 저장소가 있으면, 신규·Svelte 쪽은 **본 표를 우선**한다. |
| 모서리 | **`border-radius: 8px`** |
| 글자 | `#1a1a1a`, **13px**, `font-family: inherit` |
| 안쪽 여백 | **8px 11px** |
| placeholder | `#9ca3af`, 불투명 유지 |
| focus | **모달 구현**: 테두리 링 대신 배경만 **`#f3f1ed`** 로 전환(§7 아웃라인 포커스와 병용 가능 — 화면 성격에 맞게 하나 선택). |

**날짜 두 칸:** `.fg.fg-dates` 그리드 **1fr / 1fr**, 간격 10px · 모바일(`max-width: 560px`)에서는 1열·간격 12px, `input[type=date]` 는 **최소 높이 44px**, **font-size 16px**(iOS 줌 방지).

### 4.4 공통 버튼 스케일 (프로젝트 모달 = 기준)

| 역할 | 클래스·예시 | 스펙(요지) |
|------|-------------|------------|
| **주요 CTA(전폭)** | `.bcr` | 배경 **`#631EED`**, 글 **`#fff`**, **13px**·**굵게**, **`border-radius: 9px`**, 패딩 **10px**, 테두리 없음 |
| **주요 CTA 강조(한 구역 대표)** | `.bcr.bcr-create-project` | 같은 색 · 세로 패딩 **14px**, 글 **17px**(생성 하나만 튀게) |
| **보조(같은 폼 안 2차 액션)** | `.bcr.bcr-badge-settings` | 배경 **`#f3f1ff`**, 글 **`#2C155A`**, 호버 시 **`#e8e4ff`** |
| **세컨더리(외곽 톤·채우기 아님)** | `#BJI.proj-json-import-btn` | 배경 **`#f5f3ff`**, 글 **`#631EED`**, 호버 **`#ede9fe`** / 글 **`#5519D4`** · 전폭 · radius **9px** |
| **토글/세그먼트(둘 중 하나)** | `.nm-create-opt` / `.nm-create-opt--on` | 기본: 배경 `#e6e4ff`, 글 `#2C155A` · 선택됨: 배경 `#aaa3ff`, 글 `#fff` · **focus-visible** 시 `outline: 2px solid #631EED` |
| **작은 보조 버튼** | `.bcr.sm` | `width: auto`, 패딩 **8px 12px**, 12px 글 (초대 불러오기 등) |
| **비활성** | `.bcr:disabled` | opacity **0.55** |

다른 화면의 **보조/고스트** 버튼은 §7(보더 `#e0dbd4`, 배경 투명~`#fafaf8`)과 조합한다.

### 4.5 구역 라벨·내부 가이드 타이포

| 용도 | 클래스 | 규격 |
|------|--------|------|
| 폼 위 **섹션 머리글**(목록 위 「이 기기의 프로젝트」 등) | `.plt` | **11px**, **`#aaa`**, **대문자**, `letter-spacing: 0.05em`, **굵게** |
| 필드 위 라벨(공통 메뉴·모달에서 쓸 때) | `.fl` | **11px**, `#666`, **600**, 하단 여백 4px |
| 모달 제목 | `h3` | **15px**, `#1a1a1a`, **700** |

### 4.6 네이티브 셀렉트·체크·라디오 (표준 권장)

저장소에 `<select>` · `checkbox` 전용 스킨이 아직 없으면, 도입 시 아래를 맞춘다.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pseriesadmin/plannode](https://github.com/pseriesadmin/plannode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
