---
trigger: always_on
description: > DESIGN.md를 만들기 위한 재료를 시각적으로 결정하는 웹 애플리케이션.
---

# pre-design-md — 설계 명세서

> DESIGN.md를 만들기 위한 재료를 시각적으로 결정하는 웹 애플리케이션.

---

## 1. 개요

### 1.1 이 앱의 포지션
pre-design-md는 **디자인 산출물이 아니라 AI 입력(DESIGN.md 프롬프트)을 만드는 도구**다.

사용자는 5개 단계로 구조적 결정을 시각적으로 내린다. 앱은 그 결정을 AI 에이전트가 정확히 같은 방식으로 재현할 수 있는 프롬프트로 변환한다. 이 프롬프트를 Codex, Cursor, Codex 등에 붙여넣으면 프로젝트의 DESIGN.md가 일관된 의도를 담고 생성된다.

"또 다른 theme builder"가 아니다. 산출물이 코드가 아니라 **AI가 해석할 의도 + 값의 결합**이라는 점이 결정적 차이다.

### 1.2 비목표 (v1)
- 완전한 디자인 시스템 빌더 아님 — Figma Tokens나 Style Dictionary 대체가 아니다
- 반응형/브레이크포인트 조정 없음
- 커스텀 폰트 업로드 없음 (웹 세이프 + Google Fonts 프리셋만)
- 컴포넌트 라이브러리 생성 없음
- 다크모드는 선택사항 (v1에서는 라이트 모드 중심, 다크 지원 여부만 표기)

---

## 2. 핵심 설계 원칙

### 원칙 1 — 구조적 결정 소수화
사용자가 내리는 결정은 5단계, 각 단계에서 1~2개의 base 결정만 받는다. 나머지는 전부 자동 파생. 타이포 base size + ratio가 결정되면 type scale 전체가 자동 생성되는 식이다.

카테고리별로 수십 개 값을 개별 선택하게 하면 조합 폭발과 정합성 붕괴가 일어난다. 실제 디자인 시스템이 일관돼 보이는 이유는 "base 몇 개가 나머지를 지배"하기 때문이고, 그 구조를 그대로 UX로 가져온다.

### 원칙 2 — 프리뷰가 UX의 핵심
토큰을 추상적으로 "이거 vs 저거" 비교하게 하지 않는다. 모든 선택지는 실제 컴포넌트나 섹션에 적용된 상태로 렌더된다. "이 ratio랑 저 ratio 중에 뭐가 낫지?"는 의미 없고, "이 ratio로 그린 카드랑 저 ratio로 그린 카드 중에 뭐가 낫지?"가 의미 있다.

### 원칙 3 — 결정의 의도 보존
최종 프롬프트는 값(CSS variables)만이 아니라 **왜 이 선택인지의 자연어 의도**를 포함한다. AI 에이전트가 DESIGN.md를 읽고 실제 코드를 쓸 때, "왜 이 값인지"의 의도까지 알면 정해지지 않은 영역(새 컴포넌트, 특수 상황)에서도 일관된 판단을 할 수 있다.

---

## 3. 유저 플로우

```
Start
  │
  ▼
Step 1 — Typography     (뼈대)
  ▼
Step 2 — Spacing        (뼈대)
  ▼
Step 3 — Radius         (형태감)
  ▼
Step 4 — Shadow         (표면감)
  ▼
Step 5 — Color          (분위기)
  ▼
Preview                 (모든 토큰 적용된 완성 UI)
  ▼
Export                  (DESIGN.md 프롬프트)
```

### 플로우 규칙
- 기본은 선형(게임 캐릭터 메이커 스타일)
- 뒤 단계로 가도 이전 단계의 결정은 유지
- 이전 단계로 돌아가 값을 바꾸면 이후 단계의 프리뷰는 실시간 갱신 (단, 이후 단계에서 한 선택은 유지)
- Preview에서 만족 안 하면 특정 단계로 점프 가능
- 어느 단계에서든 Export는 가능 (현재까지 결정된 값만으로 프롬프트 생성, 단 불완전 경고)

---

## 4. 단계별 설계

각 단계는 동일한 구조를 따른다:
- **사용자 결정**: base 값 1~2개
- **자동 파생**: base로부터 생성되는 토큰 집합
- **UI 패턴**: 어떤 방식으로 시각화할지
- **Interaction 자동 파생**: color 단계 이후 hover/focus/active 등이 암묵적으로 생성됨

### 4.1 Step 1 — Typography

**사용자 결정**
- Base font size: `14 / 16 / 18 / 20px` 중 선택
- Scale ratio: `1.125 (minor second) / 1.2 (minor third) / 1.25 (major third) / 1.333 (perfect fourth) / 1.5 (perfect fifth)` 중 선택
- Font pairing 프리셋 3~5개 (예: "모던 sans", "에디토리얼 serif+sans", "테크니컬 mono+sans")

**자동 파생**
- Type scale: `xs(-2), sm(-1), base(0), md(1), lg(2), xl(3), 2xl(4), 3xl(5), 4xl(6)` → `base × ratio^n`
- Line height: 헤딩은 1.2~1.3, 본문은 1.5~1.7 (scale ratio가 클수록 본문 line-height 약간 축소)
- Font weight: 400 / 500 / 600 / 700 (pairing 프리셋에 따라 사용 폭 달라짐)

**UI 패턴**
- 가로로 나열된 프리뷰 카드. 각 카드가 하나의 (base + ratio + pairing) 조합
- 카드 안에 실제 h1~h4 + 본문 + 캡션이 전부 렌더됨 (스케일 전체가 보이도록)
- "Quick brown fox..." 류 샘플 텍스트로 읽히는 느낌을 체감
- 선택 후 다음 단계 이동 버튼 노출

### 4.2 Step 2 — Spacing

**사용자 결정**
- Base unit: `4 / 8 px` 중 선택
- Scale approach: `linear (4,8,12,16,20,24,32,40,48,64) / multiplicative (4,8,16,32,64 — 티셔츠 사이즈)` 중 선택

**자동 파생**
- Spacing tokens: `2xs, xs, sm, md, lg, xl, 2xl, 3xl, 4xl, 5xl`
- 컴포넌트별 권장 padding/gap (버튼 padding은 sm/md, 카드 padding은 lg)

**UI 패턴**
- 동일한 컴포넌트 세트(카드 + 버튼 + 리스트)가 각 spacing 프리셋으로 나란히 렌더
- 좁은 느낌 / 넉넉한 느낌의 체감이 바로 비교됨

### 4.3 Step 3 — Radius

**사용자 결정**
- Base radius: `0 (sharp) / 4 (subtle) / 8 (soft) / 12 (rounded) / 16+ (pill-like)` 중 선택
- Scale approach: `uniform (모든 컴포넌트가 동일 radius) / scaled (카드는 더 둥글게, 인풋은 덜 둥글게)`

**자동 파생**
- Radius tokens: `none, sm, md, lg, xl, full`
- 컴포넌트별 권장 값 (input=md, card=lg, badge=full 등)

**UI 패턴**
- 버튼 + 인풋 + 카드 한 세트가 각 radius 프리셋마다 렌더
- Shape이 성격을 얼마나 바꾸는지 체감

### 4.4 Step 4 — Shadow / Elevation

**사용자 결정**
- Intensity: `none / subtle / medium / strong`
- Tinted 여부: pure black shadow vs. primary 색으로 살짝 물든 shadow (primary 단계에서 결정되므로 지금은 "tinted 선호?"만 토글, 색은 Step 5 이후 반영)

**자동 파생**
- Shadow tokens: `sm, md, lg, xl` — 각 레벨은 offset + blur + opacity 조합
- Elevation 위계: dropdown < card < modal < toast

**UI 패턴**
- 동일한 카드가 각 intensity 프리셋으로 렌더
- "flat / soft / material / dramatic" 계열 체감

### 4.5 Step 5 — Color

**사용자 결정**
- Primary hue: wheel로 선택하거나 프리셋 6~8개 (파랑/청록/보라/자주/빨강/주황/노랑/초록 계열)
- Primary chroma: `muted / balanced / vivid`
- Neutral style: `pure gray / warm gray / cool gray / primary-tinted gray`
- Dark mode support: boolean

**자동 파생 (OKLCH, culori 사용)**
- Primary scale: 50 ~ 950 (11단계)
- Neutral scale: 50 ~ 950 (선택한 스타일에 따라 hue 미세 조정)
- Semantic colors: success / warning / danger / info — 각 색의 chroma를 primary의 chroma 수준에 맞춰 조정 (primary가 muted인데 semantic만 vivid면 어색함)
- **Interaction states (자동 파생, 사용자 결정 없음)**
  - hover: lightness −5%
  - active: lightness −10%
  - focus ring: `outline 2px solid var(--color-primary-500); outline-offset: 2px`
  - disabled: opacity 40%
- Tinted shadow: Step 4에서 tinted 선호면 여기서 primary hue 기반 shadow color 생성

**UI 패턴**
- 실제 UI 샘플(버튼 + 카드 + 네비 + 알럿) 위에 컬러 실시간 적용
- 팔레트 전체를 별도 영역에 노출 (이해와 검증)
- Light/Dark 토글 프리뷰 (지원 시)

### 4.6 Preview Stage

이전 단계 결정 전부 적용된 "완성된 UI"를 보여준다.

- **컴포넌트 샘플러**: 버튼(primary/secondary/ghost/destructive), 인풋(text/select/checkbox/radio), 카드, 뱃지, 알럿(info/success/warning/danger), 네비게이션
- **랜딩 섹션**: Hero(제목 + 부제 + CTA + 배경 이미지) + Feature list(3개 카드) + 심플 푸터
- **이미지**: 사전 준비된 풀에서 컬러 조화로 필터링해 자동 배치
- 만족 안 하면 단계 네비로 특정 단계 복귀

### 4.7 Export Stage

- **결정 요약**: 사람이 읽기 쉬운 형태로 모든 base 결정 나열
- **DESIGN.md 프롬프트**: 복사 버튼이 붙은 하나의 마크다운 텍스트
- **사용 가이드**: "이 프롬프트를 Codex의 `/init` 직후 또는 Cursor의 프로젝트 시작 시 붙여넣으면 DESIGN.md가 생성됩니다"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sangun-Kang/pre-design-md](https://github.com/Sangun-Kang/pre-design-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
