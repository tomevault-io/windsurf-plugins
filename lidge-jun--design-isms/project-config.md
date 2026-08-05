---
trigger: always_on
description: 49개 디자인 ism의 시각적 레퍼런스 보드와 94개 프런트엔드 UI 패턴/이펙트 카탈로그, 그리고 4개 자매 카탈로그(Color 25 / Typography 20 / Layout 25 / Motion 20). GitHub Pages 배포.
---

# AGENTS.md — Design -isms 프로젝트 가이드

## 프로젝트 개요
49개 디자인 ism의 시각적 레퍼런스 보드와 94개 프런트엔드 UI 패턴/이펙트 카탈로그, 그리고 4개 자매 카탈로그(Color 25 / Typography 20 / Layout 25 / Motion 20). GitHub Pages 배포.
각 신규 카탈로그는 `assets/data/{color,typography,layout,motion}.json` + `src/{도메인}.ts`(CatalogShell 위 어댑터) + `assets/css/{도메인}.css` + `assets/images/{도메인}/{id}/guide.png` 구조를 따르고, `scripts/verify-catalog.mjs`의 validateDomain 분기와 guide ledger(030/041·042/051/061)가 검증한다.
- **라이브**: https://lidge-jun.github.io/design-isms/
- **스택**: 정적 HTML/CSS + TypeScript source → browser JS build
- **이미지**: ima2 (`gpt-5.6-sol`, reasoning high, 1536x1024, high quality)

## 디렉토리 구조
```
701_design-isms/
├── index.html                    # 메인 페이지
├── effects.html                  # 모바일/데스크탑 UI 후보군 페이지 (94)
├── color.html                    # Color Systems 카탈로그 (25)
├── typography.html               # Typography Pairings 카탈로그 (20)
├── layout.html                   # Layout Patterns 카탈로그 (25)
├── motion.html                   # Motion Presets 카탈로그 (20)
├── AGENTS.md                     # 이 파일
├── README.md
├── assets/
│   ├── css/style.css             # 전체 스타일
│   ├── css/nav.css               # 공통 상단 메뉴 스타일
│   ├── css/effects.css           # UX 효과 페이지 전용 스타일
│   ├── css/effects-docs.css      # 효과별 장문 문서 섹션
│   ├── css/effects-demos.css     # 초기 공통 UX demo/animation
│   ├── css/effects-demos-candidates.css # 18개 legacy 비주얼 demo (46 패턴=patterns.css, WP4 신규 30개=effects-demos-expansion-*.css 6파일)
│   ├── js/app.js                 # 메인 로직 (src/app.ts build 산출물)
│   ├── js/effects-demos.js       # 효과 demo renderer (src/effects-demos.ts build 산출물)
│   ├── js/effects-docs.js        # 효과 문서 renderer (src/effects-docs.ts build 산출물)
│   ├── js/effects.js             # 효과 페이지 로직 (src/effects.ts build 산출물)
│   ├── data/isms.json            # 핵심 데이터 (49개 ism)
│   ├── data/effects.json         # 프런트엔드 UI 후보군 데이터
│   ├── data/effects-docs.json    # 효과별 배경/히스토리/사용 시점 문서
│   ├── data/research-prompts.json # Grok/ima2 프롬프트 레코드
│   └── images/
│       ├── minimalism/           # ism별 폴더
│       │   ├── landing.png
│       │   ├── shop.png
│       │   └── portfolio.png
│       ├── thumbs/               # WebP thumbnail/preview 산출물
│       │   ├── minimalism/
│       │   └── effects/
│       ├── brutalism/
│       └── ...                   # 35개 폴더
├── src/
│   ├── app.ts
│   ├── effects-demos.ts
│   ├── effects-docs.ts
│   └── effects.ts
├── structure/
│   └── README.md                 # 현재 구조와 source-of-truth 요약
├── devlog/
│   └── 260510_mobile_ux_effects/ # UI 후보군 phase docs
├── scripts/
│   └── update-isms.mjs           # JSON 업데이트 스크립트
└── .github/workflows/
    └── deploy.yml                # GitHub Pages 배포
```

---

## 현재 구현 불변 조건

<!-- data-sot:agents-counts:start -->카탈로그 source-of-truth 카운트: 49 ISMs / 94 effects / 18 FAQ answers.<!-- data-sot:agents-counts:end -->

- README, `AGENTS.md`, `structure/README.md`, `devlog/`의 설명은 실제 구현과 어긋나면 안 된다.
- 소스는 `src/*.ts`, 브라우저 산출물은 `assets/js/*.js`다. GitHub Pages가 static file을 직접 배포하므로 JS 산출물도 커밋 대상이다.
- HTML은 non-module script를 사용한다. `effects.html`은 `assets/js/effects-demos.js`를 먼저, `assets/js/effects.js`를 나중에 로드해야 한다.
- 상단 메뉴는 공개 7페이지(`index.html`, `effects.html`, `faq.html`, `color.html`, `typography.html`, `layout.html`, `motion.html`)에서 `Isms / Catalog / FAQ / GitHub / Lang / Count` 6축을 같은 순서로 유지한다. Catalog 축은 드롭다운(`src/nav-dropdown.ts`)이며 Effects / Color / Typography / Layout / Motion을 담고, 미완성 항목은 `aria-disabled`+"준비 중" 배지다. static HTML이라 공통 컴포넌트가 없으므로 7페이지를 함께 수정하고, `npm run verify:nav`가 축 순서/단일 `aria-current`(카탈로그 페이지는 트리거 소유)/드롭다운 계약/count 라벨/skip link를 검증한다.
- 페이지 전용 CSS는 inline `<style>`이 아니라 `assets/css/*.css` 파일로 둔다. FAQ는 `assets/data/faq.json` + `src/faq.ts` + `assets/css/faq.css`로 렌더링하며 `faq.html`은 thin entry 문서다.
- 셸 토큰은 `assets/css/theme-atlas.css`가 소유한다(로드 순서: `style.css` → `theme-atlas.css` → `nav.css` → 페이지 CSS). 셸 UI에 이모지 글리프를 쓰지 않는다(브랜드 마크는 `assets/icons/atlas-mark.svg`).
- `index.html`은 `assets/js/app-dialog.js`(전역 `AppDialogA11y`)와 `assets/js/app-runtime.js`(전역 `AppRuntime`)를 `assets/js/app.js`보다 먼저 로드해야 한다. `effects.html`, `faq.html`도 페이지 렌더러보다 `app-runtime.js`를 먼저 로드한다.
- 세 페이지의 안전한 storage/history 접근, loading overlay 종료, 재시도 가능한 치명 오류, 이미지 fallback은 `src/app-runtime.ts`와 `assets/css/runtime-states.css`가 공통 소유한다.
- `npm run verify`는 파일을 생성하지 않는다. `src/*.ts`를 수정하면 먼저 `npm run build`로 `assets/js/*.js`를 갱신한 뒤 verify를 실행한다.
- `data-sot:*` 마커는 `scripts/sync-sot.mjs`만 수정한다. `npm run sot:check`는 49/94/18 값을 데이터에서 유도하고, `npm run sot:sync`는 검증된 마커 내부만 원자적으로 갱신한다.
- 전체 331 PNG/WebP 쌍(211 legacy + 신규 카탈로그 120)의 해시 SoT는 `assets/data/image-pairs-manifest.json`이다. `npm run images:thumbs`가 source/preview SHA와 독립 픽셀 관계(MAE ≤18)를 기준으로 재생성 여부를 결정하고 manifest를 원자 갱신한다.
- 완성판 이미지 품질 SoT는 `091_image_quality_audit.csv`, `092_image_generation_attempts/`, immutable 093–097 baseline, 그리고 098 final sheet receipt다. `npm run verify:image-quality`는 immutable legacy 211개 슬롯과 catalog-addition 쌍(live hash), 승인된 교체, 프롬프트 provenance, 비대상 byte 안정성을 비생성 방식으로 검증한다.
- `--bootstrap-manifest`는 감사된 최초 이관 전용이며 기존 manifest가 있으면 거부된다. manifest 누락을 일반 생성으로 재신뢰하지 않는다.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lidge-jun/design-isms](https://github.com/lidge-jun/design-isms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
