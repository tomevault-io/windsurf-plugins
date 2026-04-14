---
trigger: always_on
description: This is a **Next.js 16 dog adoption platform** with a protected admin dashboard. The public site displays adoptable dogs with i18n support (English/Spanish), while admins manage dog profiles through a secure CMS.
---

# AI Coding Agent Instructions - Dog Adoption Platform

## Project Overview

This is a **Next.js 16 dog adoption platform** with a protected admin dashboard. The public site displays adoptable dogs with i18n support (English/Spanish), while admins manage dog profiles through a secure CMS.

**Stack**: Next.js 16 (App Router) • React 19 • TypeScript • Prisma • PostgreSQL • NextAuth.js 5 • Cloudinary • Tailwind CSS 4

## Critical Architecture Patterns

### Authentication Flow (NextAuth.js 5 Beta)

- **Middleware protection**: [proxy.ts](../proxy.ts) uses NextAuth's `auth` as default export with matcher for `/admin/*` routes (excludes `/admin/login`)
- **Credentials provider**: [auth.config.ts](../auth.config.ts) validates against `Admin` table using bcrypt
- **Session-based**: All admin API routes check `await auth()` before mutations
- **Login redirect**: Unauthorized admin routes → `/admin/login` (configured in `authConfig.pages`)

### Database Patterns (Prisma)

- **JSON string storage**: `Dog.gallery` is stored as `String?` in Prisma but handled as `string[]` in TypeScript
  - Parse on read: `JSON.parse(dog.gallery)` in [lib/dogs.ts](../lib/dogs.ts)
  - Stringify on write: `JSON.stringify(gallery)` in [app/api/dogs/route.ts](../app/api/dogs/route.ts)
- **Status filtering**: Public queries always filter `WHERE status != 'Adopted'` (see `getDogs()`)
- **Client singleton**: Use `prisma` from [lib/prisma.ts](../lib/prisma.ts) - prevents hot reload connection issues

### Image Upload Flow (Cloudinary)

1. Admin selects image via [DogForm](../app/admin/components/DogForm.tsx) → shows crop modal
2. [ImageCropModal](../app/admin/components/ImageCropModal.tsx) allows resize, crop, rotate, and quality adjustment
3. Cropped image converted to blob → `POST /api/upload`
4. [route.ts](../app/api/upload/route.ts) converts File to Buffer → uploads to Cloudinary under `dogs/[dogName]/` folder
5. Returns Cloudinary URL (e.g., `https://res.cloudinary.com/.../dogs/buddy/img.webp`)
6. URLs stored in database, not files in `/public`

- **No local file storage**: All images go to Cloudinary CDN, never commit images to Git
- **Client-side cropping**: Uses `react-easy-crop` library for interactive image editing
- **Image utilities**: [lib/image-utils.ts](../lib/image-utils.ts) handles cropping, rotation, and resizing
- **Automatic resizing**: Default max width 1200px, quality 90% (adjustable in crop modal)

### i18n Implementation (Cookie-Based)

- **Server-side locale detection**: [lib/i18n-server.ts](../lib/i18n-server.ts) checks cookie → `Accept-Language` header → defaults to "en"
- **Message structure**: Flat JSON in [lib/i18n-messages.ts](../lib/i18n-messages.ts) with dot notation keys (e.g., `"hero.title"`)
- **Client hydration**: [LocaleProvider](../app/i18n/LocaleProvider.tsx) wraps app with React Context for `useI18n()` hook
- **Switching**: [LanguageSwitcher](../app/components/LanguageSwitcher.tsx) calls server action `setLocaleCookie()` → revalidates path

### Validation Strategy (Zod)

- **Shared schemas**: [lib/validations.ts](../lib/validations.ts) exports `dogSchema`, `dogUpdateSchema`, `loginSchema`
- **Used in**: API routes (`.parse()` throws 400), form submissions, TypeScript types via `z.infer<>`
- **Enum enforcement**: `sex`, `size`, `status` use Zod enums matching Prisma schema exactly

## Development Workflows

### Local Development Setup

```bash
pnpm install                          # Install dependencies
npx prisma migrate dev --name init    # Create database + run migrations
npx prisma generate                   # Generate Prisma Client
pnpm dev                              # Start dev server (localhost:3000)
```

### Database Operations

```bash
npx prisma studio                     # Open database GUI (localhost:5555)
npx prisma migrate dev --name <name>  # Create new migration
npx prisma db seed                    # Run seed file (seeds default admin + 9 dogs)
npx prisma migrate reset              # ⚠️ Nuke database and re-seed
```

### Admin Credentials

- **Default credentials** (must change before deployment):
  - Email: `admin@example.com`
  - Password: `password123`
- Set in `.env`: `ADMIN_EMAIL` and `ADMIN_PASSWORD` (bcrypt hashed in seed)
- **⚠️ SECURITY**: Always change default credentials before deploying to production
- Seed script ([prisma/seed.ts](../prisma/seed.ts)) creates admin user from env vars

## Documentation Maintenance

**CRITICAL**: When adding, modifying, or removing features, AI agents MUST update relevant documentation:

### Always Update When:

- **Adding new features**: Update README.md, add to Feature Roadmap section below
- **Changing architecture**: Update this file and [documentation/ARCHITECTURE.md](../documentation/ARCHITECTURE.md)
- **Modifying API routes**: Document in [documentation/ARCHITECTURE.md](../documentation/ARCHITECTURE.md) data flow sections
- **Adding database models**: Update [prisma/schema.prisma](../prisma/schema.prisma) and document in Database Patterns section above
- **Changing environment variables**: Update deployment section and create/update `.env.example`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/demaceo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
