---
trigger: always_on
description: Always prefer Serena's symbolic tools (find_symbol, get_symbols_overview, replace_symbol_body, etc.) over grep, read_file, and text-based edits for code search and modification.
---

# CLAUDE.md

## Tool Usage
Always prefer Serena's symbolic tools (find_symbol, get_symbols_overview, replace_symbol_body, etc.) over grep, read_file, and text-based edits for code search and modification.
This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands

- `bun run dev` — Next.js dev server (Turbopack)
- `bun run build` — `prisma generate` + `next build`
- `bun run type-check` — `tsc --noEmit`
- `bun run check` — Biome lint + format combined
- `bun run check:fix` — Auto-fix Biome issues
- Before pushing: `bun run type-check && bun run check`
- Prisma: `bunx prisma migrate dev -n "<msg>"`, `bunx prisma generate`, `bunx prisma studio`
- Pre-commit hook (Husky): `biome check --write --no-errors-on-unmatched` via lint-staged
- **No test framework** — no unit/integration test setup exists yet

## Tech Stack

- **Next.js 16** (App Router) + **React 19** + **TypeScript** (strict) + **Bun**
- **PostgreSQL** via **Prisma** ORM (Neon), **Iron Session** (cookie auth, 7-day expiry)
- **Biome v2** for lint/format — line width 80, single quotes (JS), double quotes (JSX), auto-organizes imports
  - Key rules: `noUnusedImports: warn`, `noExplicitAny: warn`, `useImportType: warn`, `noUnusedVariables: warn`, `useExhaustiveDependencies: warn`
- **Tailwind CSS v3** + **shadcn/ui** (Radix) + **CVA**
- **TanStack Query** (client state — `refetchOnFocus: false`, `retry: 3`), **React Hook Form** + **Zod** (validation)
- **TipTap** (rich text), **@dnd-kit** (drag-drop), **react-day-picker** (calendar)
- **Cloudflare Images/Stream** (media hosting), **PortOne V2** (payments)
- **Resend** (email), **Aligo/Solapi** (SMS — provider via `SMS_PROVIDER` env)
- Deployed on **Vercel**

## Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (home)/             # 홈페이지 — 단일 page.tsx
│   ├── (content)/          # Public 콘텐츠 흐름 (programs, journal, drops, archive, discover, tickets/order)
│   ├── (page)/             # Static info + entity directory (about, terms, privacy, refund-policy, partnership, artists, artworks, venues, forms)
│   ├── (auth)/             # 인증 + admin 그룹 (admin layout 포함)
│   ├── api/                # API routes
│   └── scan/               # QR 외부 카메라 fallback (/scan/[token])
├── modules/<domain>/       # Feature modules
│   ├── server/actions.ts   # Server actions ('use server')
│   └── ui/
│       ├── views/          # 페이지 단위 큰 컴포넌트 (DropDetailView, FormBuilderView 등)
│       └── components/     # 작은 building block — form, card, list, section 등 모두 (※ 별도 section 폴더 없음, 통합됨)
├── components/             # 공용 UI
│   ├── ui/                 # shadcn primitives (Button, Dialog, ConfirmDialog 등)
│   ├── admin/              # admin 공통 (AdminNav, DeleteButton 등)
│   ├── layout/             # 사이트 layout (Header, Footer, PublicHeader, LegalPageLayout)
│   ├── shared/             # 작은 공용 utility (BackButton, FadeIn, FilterChip 등)
│   ├── icons/              # SVG 아이콘
│   ├── media/              # 미디어 표시 (CloudflareStreamVideo, MediaGallery, SortableMediaList)
│   ├── image/              # 이미지 표시·업로드 박스 (CarouselGallery, MultiImageBox, SingleImageBox)
│   ├── rich-editor/        # TipTap 에디터 — journal/drops에서 공유
│   └── seo/                # JSON-LD schema 컴포넌트
├── hooks/                  # 공용 커스텀 훅 (use-toast, use-single-image-upload 등)
├── lib/                    # 인프라 + 유틸 (auth, cdn, db, email, payment, pnl, sms, schemas, utils)
└── middleware.ts            # Route protection
```

**Import paths**: `@/*` → `./src/*`. **Filenames**: kebab-case. **UI language**: Korean.

### 폴더링 컨벤션

**`components/<topic>/` vs `modules/<x>/ui/components/`**:
- 여러 모듈에서 공유하는 컴포넌트는 `components/<topic>/`. 단일 모듈 전용은 `modules/<x>/ui/components/`.
- 예: `email-editor`는 email 모듈만 쓰니 `modules/email/ui/components/email-editor/`. `rich-editor`는 journal+drops 둘 다 쓰니 `components/rich-editor/`.

**`(content)` vs `(page)` 라우트 그룹**:
- `(content)`: 사이트의 콘텐츠 흐름(브랜드 메인 동선) — programs, journal, drops, archive, discover.
- `(page)`: 디렉토리/static info — entity 목록·상세(artists, artworks, venues, forms), 약관(terms, privacy, refund-policy).
- 굳이 분리할 가치가 모호하면 같은 그룹에 둬도 OK. 분류는 navigation 흐름 기준이지 보안 기준이 아님.

**`lib/utils/`** — 토픽별 분리:
- `cn.ts`(Tailwind), `date.ts`, `image-url.ts`, `upload.ts`, `text.ts`, `ticket-status.ts`, `ticket-token.ts`, `bank-transfer.ts`, `media-upload.ts`
- `index.ts`에서 모두 re-export → 호출자는 `@/lib/utils`로 import

### Modules

`programs`, `journal`, `artists`, `venues`, `artworks`, `auth`, `forms`, `sms`, `email`, `drops`, `tickets`, `home`, `estimates`, `pnl` — each with `server/actions.ts` + `ui/{views,components}/`. Query provider: `src/modules/providers.tsx`.

## Key Patterns

### Server Actions
- `'use server'` directive, return `{ success, error? }` or `{ success, data? }`
- Admin guard: `requireAdmin()` → `{ success, userId }`
- Auth check: `canManage(sessionId, authorId?)` — ADMIN or resource owner
- **Important**: `'use server'` files cannot export synchronous functions — move utility functions to `@/lib/utils`

### React 19 + Radix UI Compatibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamkimseunghyun/prectxe-site](https://github.com/iamkimseunghyun/prectxe-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
