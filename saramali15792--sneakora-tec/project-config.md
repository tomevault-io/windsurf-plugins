---
trigger: always_on
description: *An authoritative reference governing all work performed on this full-stack e-commerce application.*
---

# Constitution of the Sneakora Project

*An authoritative reference governing all work performed on this full-stack e-commerce application.*

---

## Preamble

This document establishes the rules, principles, and procedures by which any agent shall operate when contributing to the Sneakora project. All agents are bound to follow the articles herein. No deviation shall be made without explicit instruction from the project owner.

---

## Article I — Workflow & Procedure

### Section 1.1 — Mandatory Execution Order

Every agent, before writing any code, shall adhere to the following sequence:

1. **Check Skills First.** Before writing any code, the agent shall load the matching skill via the `skill` tool. Skills reside in `.opencode/skills/` (project-local). If a skill covers the task at hand, its instructions shall be followed. Skill names and paths shall not be hardcoded — they shall be referenced dynamically by name. A complete catalog of installed skills is maintained in Article VIII.

2. **Research via Context7 MCP.** For any question concerning a framework, library, or language — including but not limited to Next.js, React, BetterAuth, Prisma, PostgreSQL, Tailwind CSS, and shadcn/ui — the agent shall call `context7_resolve-library-id` followed by `context7_query-docs` to obtain authoritative documentation and code examples. The MCP server is pre-configured in `opencode.json` with an active API key.

3. **Read Existing Code.** Before writing any new code, read the relevant existing files to match patterns, naming conventions, and architectural decisions.

4. **Implement Following Conventions.** All new code shall match the patterns established in this document and the target architecture defined in Article III.

5. **Verify Changes.** After implementation, verify the application works as expected. For Next.js, run `npm run dev` and check the application in the browser. For backend changes, restart the development server.

### Section 1.2 — Project Transformation Status

This project is in the process of being **completely rebuilt** from a vanilla HTML/CSS/JS + Express.js + MySQL stack into a modern production-ready e-commerce platform. The old codebase remains on disk for reference but **shall not be modified** — all new work targets the new architecture described below.

---

## Article II — Project Identity & Purpose

### Section 2.1 — Nature of the Application

Sneakora is a full-stack e-commerce store being rebuilt as a modern, production-ready platform. The new stack uses **Next.js 14+** (React with TypeScript) on the frontend, **BetterAuth** for authentication, **Prisma** as the ORM, **Neon** (serverless PostgreSQL) as the database, and is deployed on **Vercel**.

### Section 2.2 — Business Capabilities

The application targets a **full premium e-commerce suite** with the following capabilities:

| Category | Features |
|----------|----------|
| **Core Shopping** | Product catalog by category (Men, Women, Kids, Sports, Casual), product detail pages, search & filter, shopping cart, checkout |
| **User Features** | Registration, login, profile management, order history, wishlist, recently viewed |
| **Premium Features** | Reviews & ratings, size guide, coupon/discount system, live chat, blog |
| **Admin** | Product CRUD, order management, user management, analytics dashboard |
| **Engagement** | Newsletter signup, contact form, about us page, social media integration |

### Section 2.3 — Design Direction

The visual design follows a **Modern-Sporty** aesthetic — clean, energetic, bold typography, action-oriented design language inspired by leading sportswear brands (Nike, Adidas). Dark theme with neon accent colors, glassmorphism, micro-animations, and 3D elements.

---

## Article III — Architecture (Target)

### Section 3.1 — Frontend Layer

The frontend is built with **Next.js 14+** using the App Router, TypeScript, Tailwind CSS, and shadcn/ui components.

```
frontend/
├── app/                           # Next.js App Router pages
│   ├── layout.tsx                 # Root layout with providers
│   ├── page.tsx                   # Homepage
│   ├── shop/
│   │   ├── page.tsx               # Catalog / category listing
│   │   └── [id]/page.tsx          # Product detail page
│   ├── cart/page.tsx              # Shopping cart
│   ├── checkout/page.tsx          # Checkout flow
│   ├── profile/
│   │   ├── page.tsx               # User profile
│   │   └── orders/page.tsx        # Order history
│   ├── wishlist/page.tsx          # User wishlist
│   ├── admin/
│   │   ├── page.tsx               # Admin dashboard
│   │   ├── products/page.tsx      # Product management
│   │   └── orders/page.tsx        # Order management
│   ├── blog/
│   │   ├── page.tsx               # Blog listing
│   │   └── [slug]/page.tsx        # Blog post
│   ├── about/page.tsx             # About us
│   ├── contact/page.tsx           # Contact form
│   └── api/                       # API routes (Next.js route handlers)
├── components/
│   ├── ui/                        # shadcn/ui base components
│   ├── layout/                    # Navbar, Footer, Sidebar
│   ├── product/                   # ProductCard, ProductGallery, ProductInfo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SARAMALI15792/Sneakora_tec](https://github.com/SARAMALI15792/Sneakora_tec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
