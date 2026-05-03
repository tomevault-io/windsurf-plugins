---
trigger: always_on
description: This file provides guidance to AI assistants (Claude, GPT, etc.) when working with code in this repository.
---


# CLAUDE.md

This file provides guidance to AI assistants (Claude, GPT, etc.) when working with code in this repository.

## Project Overview

**AncestorTree** (Gia Phả Điện Tử) is a digital family tree management system for Chi tộc Đặng Đình, Thạch Lâm, Hà Tĩnh.

- **Repository:** https://github.com/Minh-Tam-Solution/AncestorTree
- **Current Version:** v2.5.0 (Cộng đồng & Nâng cao — Feed, Search, GEDCOM 7.0, Notifications, Nhà thờ họ, SEO)
- **SDLC Tier:** LITE (5 stages)
- **Tech Stack:** Next.js 16, React 19, Tailwind CSS 4, Supabase, Electron 34 (desktop)
- **Built with:** [TinySDLC](https://github.com/Minh-Tam-Solution/tinysdlc) + [MTS-SDLC-Lite](https://github.com/Minh-Tam-Solution/MTS-SDLC-Lite)

## SDLC Framework v6.1.0 - LITE Tier

This project follows MTS SDLC Framework with 5 stages:

```
docs/
├── 00-foundation/     # Vision, scope, requirements, community
│   └── 06-Community/  # Community launch posts (7 platform-specific)
├── 01-planning/       # Roadmap, sprints, milestones
├── 02-design/         # Architecture, UI/UX, data models
├── 04-build/          # Implementation guidelines
└── 05-test/           # Test plans, QA
```

**DO NOT** use generic 6-stage or 11-stage SDLC structure.
**ALWAYS** use the structure defined in `.sdlc-config.json`.

## File Header Standard

All documentation files MUST include YAML front matter:

```yaml
---
project: AncestorTree
path: docs/XX-stage/filename.md
type: document-type
version: X.X.X
updated: YYYY-MM-DD
owner: team/person
status: draft|review|approved
---
```

All code files MUST include header comment:

```typescript
/**
 * @project AncestorTree
 * @file src/path/to/file.ts
 * @description Brief description
 * @version 1.0.0
 * @updated 2026-02-25
 */
```

## Project Structure

```
AncestorTree/
├── docs/                           # SDLC Documentation
│   ├── 00-foundation/              # Vision, requirements
│   │   └── 06-Community/           # Community launch posts
│   ├── 01-planning/                # Sprints, roadmap
│   ├── 02-design/                  # Architecture, UI/UX
│   ├── 04-build/                   # Implementation
│   └── 05-test/                    # Testing
├── frontend/                       # Next.js application
│   ├── src/app/                    # App router (route groups)
│   │   ├── (auth)/                 # Auth pages (login, register, forgot-password, reset-password, pending-verification)
│   │   └── (main)/                 # Main app with sidebar
│   │       ├── achievements/       # Vinh danh thành tích (Sprint 6)
│   │       ├── cau-duong/          # Lịch Cầu đường (Sprint 7)
│   │       ├── charter/            # Hương ước (Sprint 6)
│   │       ├── contributions/      # Đóng góp (Sprint 4)
│   │       ├── directory/          # Thư mục thành viên (Sprint 4)
│   │       ├── documents/book/     # Gia phả sách (Sprint 5)
│   │       ├── documents/library/ # Kho tài liệu (Sprint 11)
│   │       ├── events/             # Lịch sự kiện (Sprint 4)
│   │       ├── feed/              # Góc giao lưu (Sprint 15)
│   │       ├── fund/               # Quỹ khuyến học (Sprint 6)
│   │       ├── help/               # Hướng dẫn sử dụng (Sprint 11)
│   │       ├── notifications/     # Thông báo (Sprint 16)
│   │       ├── people/             # Quản lý thành viên
│   │       ├── relationship/      # Tìm quan hệ (Sprint 13)
│   │       ├── settings/profile/   # Hồ sơ tài khoản (Sprint 12)
│   │       ├── settings/security/  # Bảo mật MFA (Sprint 12)
│   │       ├── stats/             # Thống kê (Sprint 13)
│   │       ├── tree/               # Cây gia phả
│   │       └── admin/              # Admin panel
│   │           ├── achievements/   # QL Vinh danh (Sprint 6)
│   │           ├── cau-duong/      # QL Cầu đường (Sprint 7)
│   │           ├── charter/        # QL Hương ước (Sprint 6)
│   │           ├── contributions/  # QL Đóng góp (Sprint 4)
│   │           ├── backup/         # Sao lưu & Phục hồi (Sprint 12)
│   │           ├── documents/      # QL Tài liệu (Sprint 11)
│   │           ├── duplicates/    # QL Trùng lặp (Sprint 14)
│   │           ├── feed/          # QL Bài viết (Sprint 15)
│   │           ├── fund/           # QL Quỹ & Học bổng (Sprint 6)
│   │           ├── registrations/ # QL Đơn ghi danh (Sprint 18)
│   │           ├── settings/       # Cài đặt dòng họ
│   │           └── users/          # QL Người dùng (verify/suspend/delete + bulk actions)
│   ├── src/components/             # React components
│   │   ├── ui/                     # shadcn/ui components
│   │   ├── auth/                   # Auth components (auth-provider, verification-guard)
│   │   ├── layout/                 # Layout components (sidebar, header)
│   │   ├── feed/                   # Feed components (post-card, compose-box, comments)
│   │   ├── home/                   # Homepage components (featured-charter)
│   │   └── people/                 # People components (person-form, family-relations-card)
│   ├── src/hooks/                  # Custom React hooks
│   │   ├── use-achievements.ts     # Achievement CRUD hooks (Sprint 6)
│   │   ├── use-cau-duong.ts        # Cầu đường rotation hooks (Sprint 7)
│   │   ├── use-clan-articles.ts    # Charter CRUD hooks (Sprint 6)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Minh-Tam-Solution/AncestorTree](https://github.com/Minh-Tam-Solution/AncestorTree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
