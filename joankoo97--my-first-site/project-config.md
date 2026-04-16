---
trigger: always_on
description: > 이 문서는 AI (ChatGPT, Claude, Gemini 등)가 이 프로젝트를 빠르게 이해하고 수정할 수 있도록 작성되었습니다.
---

# 🤖 AI를 위한 프로젝트 구조 설명서

> 이 문서는 AI (ChatGPT, Claude, Gemini 등)가 이 프로젝트를 빠르게 이해하고 수정할 수 있도록 작성되었습니다.

## 📁 프로젝트 개요

- **프로젝트 타입**: Next.js 15 인라인 편집 포트폴리오 템플릿
- **주요 기술**: Next.js 15, TypeScript, Tailwind CSS 3.4, Lucide React, next-themes
- **편집 방식**: 컴포넌트별 인라인 편집 + 로컬/파일 저장 API 제공
- **목적**: 비개발자도 쉽게 수정 가능한 한국어 포트폴리오 템플릿
- **제작자**: 백상 (@Mrbaeksang95) - https://devcom.kr/

## 🎯 핵심 특징

1. **인라인 편집 중심**: `Ctrl + E` 또는 우하단 버튼으로 편집 모드 전환, `useInlineEditor` 훅이 로컬 스토리지와 API 저장을 모두 관리
2. **자동 자산 관리**: `/app/api/upload-*` 라우트가 의미있는 파일명 생성과 기존 파일 정리를 담당 (개발 모드 전용)
3. **섹션 단위 데이터 모델**: Hero/About/Projects/Contact/Footers 등 각 컴포넌트에 `defaultInfo` 및 저장 키가 정의되어 있어 빈 값이면 자동으로 비표시
4. **완전한 반응형 & 테마**: Tailwind CSS 유틸리티와 `next-themes`로 다크/라이트 + 디바이스별 레이아웃 지원
5. **한글 친화 기능**: QR 코드(vCard 3.0), OG 이미지 API, Lucide 아이콘 한글 안내 등 한국어 설명이 기본 제공

## 📂 폴더 구조

```
vibeCoding/
├── app/
│   ├── api/
│   │   ├── delete-image/        # 업로드 자산 정리
│   │   ├── og-image/            # OG 이미지 생성
│   │   ├── update-component/    # 전체 컴포넌트 데이터 저장
│   │   ├── update-field/        # 단일 필드 저장
│   │   ├── upload-image/        # 이미지 업로드 (dev only)
│   │   └── upload-video/        # 비디오 업로드 (dev only)
│   ├── globals.css              # Tailwind 기반 전역 스타일
│   ├── layout.tsx               # 루트 레이아웃 및 메타데이터
│   ├── page.tsx                 # 홈 구성 (Hero~Footer)
│   └── safelist.css             # Tailwind safelist 용도
├── components/
│   ├── editable/                # 인라인 편집 전용 컴포넌트군
│   │   ├── editable-background.tsx
│   │   ├── editable-icon.tsx
│   │   ├── editable-list.tsx
│   │   ├── editable-media.tsx
│   │   └── editable-text.tsx
│   ├── ui/                      # 버튼/카드/토스트 등 공용 UI
│   ├── about.tsx                # 소개 섹션 + 타임라인/스킬 편집
│   ├── contact.tsx              # 연락처 + QR/소셜 편집
│   ├── footer.tsx               # 푸터 및 네비 동기화
│   ├── global-save-button.tsx   # 파일 저장 토글 버튼
│   ├── header.tsx               # 상단 헤더 + 고정 네비게이션
│   ├── hero.tsx                 # 히어로 섹션 + 배경/소셜 편집
│   ├── navbar.tsx               # 섹션 네비게이션 메뉴
│   ├── projects.tsx             # 프로젝트 카드/미디어 관리
│   ├── save-field-button.tsx    # 단일 필드 저장 버튼
│   ├── save-to-file-button.tsx  # JSON 파일 저장 버튼
│   ├── site-title.tsx           # 상단 타이틀/설명 편집
│   ├── theme-provider.tsx       # 테마 컨텍스트 연결
│   └── theme-toggle.tsx         # 라이트/다크 토글
├── contexts/
│   └── inline-editor-context.tsx # 편집 상태 + 저장 로직
├── hooks/
│   └── use-toast.ts             # 토스트 훅 (UI 공유)
├── lib/
│   ├── cleanup-storage.ts       # 로컬 파일 정리 유틸
│   ├── constants.ts             # 공용 상수/스타일
│   ├── metadata.ts              # site metadata
│   ├── reset-images.ts          # 이미지 리셋 유틸
│   └── utils.ts                 # className 병합 등
├── public/
│   └── uploads/                 # 업로드된 미디어 (개발 시 생성)
├── publicuploads/               # 서버 업로드 보조 경로 (개발용)
├── components.json              # 컴포넌트 저장 데이터 (dev)
├── GEMINI.md
├── README.md
├── package.json
└── tsconfig.json
```

## 🔧 각 파일별 수정 방법

### ⭐ 주요 섹션 기본 데이터

> 인라인 편집 기능으로 런타임에 대부분의 값을 바꿀 수 있지만, **새 프로젝트의 기본값**이나 **필드 구성**을 조정할 때는 각 컴포넌트의 `default*` 상수를 수정하세요. 모든 컴포넌트는 `useInlineEditor` 컨텍스트를 통해 `saveData(key, value)` / `saveToFile(component, field, value)`를 호출합니다.

#### 1. `components/hero.tsx`
- `defaultInfo`: 인삿말(`greeting`), 이름(`name`), 직함(`title`), 소개문(`description`), 배경(`background`), CTA 버튼(`projectButton`) 기본값을 정의합니다.
- `defaultSocialLinks`: `{ name, icon, url }` 배열. 사용할 수 있는 아이콘 키는 `AVAILABLE_ICONS` 객체에 정리되어 있습니다.
- 저장 키: `hero-info`, `hero-social-links`, `hero-background`.
```typescript
const defaultInfo = {
  greeting: "안녕하세요,",
  name: "당신의 이름",
  title: "프론트엔드 개발자",
  description: "창의적인 아이디어로 웹 경험을 디자인합니다.",
  background: { image: "", video: "", color: "", opacity: 0.1 }
}
```

#### 2. `components/about.tsx`
- `defaultInfo` 안에 `experienceCards`, `skills`, `story`, `hobbies`, `background`가 한 번에 정의됩니다.
- 카드/스킬 추가 버튼이 있으므로 기본 배열의 최대 길이는 자유롭게 조정할 수 있습니다.
- 저장 키: `about-info`, `about-background`.
```typescript
const defaultInfo = {
  experienceCards: [{ icon: "briefcase", title: "회사명", period: "2020 - 현재", description: "직무" }],
  skills: [{ icon: "code", title: "프론트엔드", description: "React, TypeScript" }],
  story: ["당신의 이야기를 입력하세요"],
  hobbies: ["📚 독서"],
  background: { image: "", video: "", color: "", opacity: 0.1 }
}
```

#### 3. `components/projects.tsx`
- `defaultInfo`에 `initialDisplay`, `loadMoreCount`, `projects` 배열을 정의합니다.
- `projects` 항목은 `{ image, video?, title, description }` 형태이며 비어 있으면 편집 UI에서 새로 추가합니다.
- 저장 키: `projects-info`, `projects-background`. `saveToFile('projects', 'Info', data)`를 호출하므로 필드명 주의.

#### 4. `components/contact.tsx`
- `defaultInfo`는 연락처, 소개 문구, QR 설정(`qrContent` 배열) 등을 포함합니다.
- `defaultSocialLinks` 배열을 변경하면 초기 소셜 버튼 구성이 달라집니다. 아이콘 키는 `AVAILABLE_ICONS` 참고.
- 저장 키: `contact-info`, `contact-social-links`, `contact-background`.

#### 5. `components/footer.tsx`
- `defaultInfo`에 푸터 표시 여부, 빠른 링크, 연락처 영역 텍스트가 들어 있습니다.
- `templateCreator`/`showTemplateCredit` 값은 고정이므로 수정하려면 로직을 변경해야 합니다.
- 헤더에서 `nav-config` 데이터를 읽어와 푸터 빠른 링크를 동기화합니다.

#### 6. `components/header.tsx`
- `defaultConfig`에 로고 텍스트/이미지, 네비게이션 항목(`items` 배열), 사이트 타이틀/설명이 포함됩니다.
- 아이콘 이름은 `ICON_MAP` 키(대문자 시작)와 매핑되어 있으므로 문자열 그대로 저장하면 자동으로 변환됩니다.
- 저장 키: `nav-config`. `saveToFile('navigation', 'Config', data)` / `saveFieldToFile('site-title', value)` 사용.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joankoo97) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
