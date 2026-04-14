---
trigger: always_on
description: Listings Marketplace – GitHub Copilot Instructions
---

Listings Marketplace – GitHub Copilot Instructions
Project Overview
Project Name: Listings Marketplace (Обяви с картинки)
Goal: A multi-page, full-stack web application where users can post, browse, and manage listings with image uploads. Admins can moderate content. Built as a Soft-Tech-with-AI Capstone Project.

Tech Stack
LayerTechnologyFrontendVanilla JavaScript (ES Modules), HTML5, CSS3UI FrameworkBootstrap 5 (CDN or npm)Build ToolVite (multi-page mode)Backend / BaaSSupabaseDatabasePostgreSQL (via Supabase)AuthenticationSupabase Auth (email/password)File StorageSupabase StorageAPISupabase REST API + JS SDK (@supabase/supabase-js)HostingNetlifySource ControlGitHubAI AssistantGitHub Copilot inside VS Code

Strict Rules – Follow at All Times

No frameworks – Do NOT use React, Vue, Angular, Svelte, or TypeScript. Vanilla JS only.
Multi-page architecture – Every page is a separate .html file. Do NOT build a SPA with client-side routing.
Modular JS – Each page loads its own src/pages/*.js module. Shared logic lives in src/services/, src/components/, or src/utils/.
Security first – ALL data access rules are enforced via Supabase Row-Level Security (RLS) policies in the database. Never rely on UI-only checks for security.
Admin logic belongs in RLS + DB – The is_admin() SQL function (security definer) must be the single source of truth for admin checks.
All DB calls go through src/services/ – Pages never import supabaseClient directly for queries; they call service functions.
Environment variables – Supabase URL and anon key are stored in .env (Vite prefix VITE_). Never hardcode credentials.
Mobile-first – Use Bootstrap grid and utility classes. Every page must be usable on a 375px screen.
Consistent error handling – All async service calls are wrapped in try/catch. Errors surface via src/components/toast.js.


Folder Structure
listings-marketplace/
├── .github/
│   └── copilot-instructions.md     ← this file
├── .env                             ← VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY
├── .env.example
├── .gitignore
├── vite.config.js                   ← multi-page input config
├── package.json
├── README.md
│
├── index.html                       ← Public listing feed + search/filter
├── register.html                    ← User registration
├── login.html                       ← User login
├── listing-create.html              ← Create new listing + image upload
├── listing-edit.html                ← Edit listing + manage images
├── listing-details.html             ← Listing detail + comments + image download
├── profile.html                     ← User profile + avatar upload
├── admin.html                       ← Admin panel (users + listings moderation)
│
└── src/
    ├── pages/
    │   ├── index.js
    │   ├── register.js
    │   ├── login.js
    │   ├── listing-create.js
    │   ├── listing-edit.js
    │   ├── listing-details.js
    │   ├── profile.js
    │   └── admin.js
    │
    ├── services/
    │   ├── supabaseClient.js        ← createClient() singleton
    │   ├── authService.js           ← register, login, logout, getSession
    │   ├── listingsService.js       ← CRUD for listings
    │   ├── commentsService.js       ← CRUD for comments
    │   ├── storageService.js        ← upload/download/delete images & avatars
    │   ├── profileService.js        ← read/update profile, avatar
    │   └── adminService.js          ← admin: user list, change roles, moderate
    │
    ├── components/
    │   ├── navbar.js                ← renders nav, highlights active page, auth state
    │   ├── toast.js                 ← showToast(message, type)
    │   └── loader.js                ← showLoader() / hideLoader()
    │
    ├── utils/
    │   ├── guard.js                 ← requireAuth(), requireAdmin(), redirectIfLoggedIn()
    │   └── format.js                ← formatDate(), formatPrice(), truncate()
    │
    └── styles/
        └── app.css                  ← global overrides on top of Bootstrap

Database Schema
Table: profiles
sqlid          uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE
display_name text
avatar_url  text
created_at  timestamptz DEFAULT now()
Table: user_roles
sqluser_id  uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE
role     text NOT NULL CHECK (role IN ('user', 'admin')) DEFAULT 'user'
Table: listings
sqlid          uuid PRIMARY KEY DEFAULT gen_random_uuid()
owner_id    uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE
title       text NOT NULL
description text
price       numeric(10, 2)
status      text NOT NULL CHECK (status IN ('draft', 'published', 'archived')) DEFAULT 'draft'
created_at  timestamptz DEFAULT now()
Table: listing_images
sqlid          uuid PRIMARY KEY DEFAULT gen_random_uuid()
listing_id  uuid NOT NULL REFERENCES listings(id) ON DELETE CASCADE
owner_id    uuid NOT NULL REFERENCES auth.users(id)
file_path   text NOT NULL
public_url  text NOT NULL
created_at  timestamptz DEFAULT now()
Table: comments
sqlid          uuid PRIMARY KEY DEFAULT gen_random_uuid()
listing_id  uuid NOT NULL REFERENCES listings(id) ON DELETE CASCADE
author_id   uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE
content     text NOT NULL
created_at  timestamptz DEFAULT now()
ER Diagram (Mermaid)
mermaiderDiagram
    profiles {
        uuid id PK

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DMY76657) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
