---
trigger: always_on
description: This guide covers all server-side implementation details for Vibe Bar, including Supabase database setup, authentication, OpenRouter AI integration, and Next.js Server Actions.
---

# Vibe Bar - Backend Implementation Guide

## 🗄️ Backend Architecture Overview

This guide covers all server-side implementation details for Vibe Bar, including Supabase database setup, authentication, OpenRouter AI integration, and Next.js Server Actions.

# Vibe Bar - Backend Implementation Guide

## 📊 Current Backend Status

**✅ IMPLEMENTED:**
- FastAPI application with OpenRouter AI integration
- Pydantic models for request/response validation
- CORS middleware for frontend communication
- Environment configuration management
- Recipe generation endpoint working
- Health check and test endpoints

**🚧 PHASE 2 (To Be Implemented):**
- Database integration (Supabase recommended)
- User authentication and authorization
- Recipe saving and management endpoints
- User profile management
- Rate limiting and usage tracking

**📋 FUTURE PHASES:**
- Advanced AI features and model switching
- Recipe sharing and social features
- Performance optimizations and caching
- Production deployment configuration

---

### Current Backend Tech Stack ✅
- **API Framework**: FastAPI (Python)
- **AI Integration**: OpenRouter.ai API
- **Data Validation**: Pydantic models
- **Configuration**: Environment variables with python-dotenv

### Planned Backend Tech Stack (Phase 2)
- **Database**: Supabase PostgreSQL
- **Authentication**: Supabase Auth
- **Server Logic**: FastAPI + Supabase integration
- **File Storage**: Supabase Storage (future)
- **Real-time**: Supabase Realtime (future)

---

## 🎯 Current Backend Implementation

### FastAPI Backend Structure ✅ (IMPLEMENTED)
```
backend/
├── app/
│   ├── main.py                # FastAPI application entry point
│   ├── config.py              # Environment configuration
│   ├── models/                # Pydantic data models
│   │   ├── common.py          # Base response models
│   │   └── cocktail.py        # Cocktail-specific models
│   └── services/              # Business logic services
│       └── openrouter.py      # AI integration service
├── requirements.txt           # Python dependencies
├── test_*.py                 # API testing scripts
└── .env                      # Environment variables
```

### Database Setup (Phase 2 - To Be Implemented)

#### Recommended: Supabase Integration
```
supabase/
├── config.ts              # Supabase client configuration
├── migrations/             # Database migrations
│   ├── 20240101000001_initial_schema.sql
│   ├── 20240101000002_profiles_table.sql
│   ├── 20240101000003_recipes_tables.sql
│   └── 20240101000004_rls_policies.sql
├── seed.sql               # Initial data seeding
└── functions/             # Edge Functions (future)
    └── generate-recipe/
        └── index.ts
```

### Database Schema & Migrations

#### Initial Schema (`migrations/20240101000001_initial_schema.sql`)
```sql
-- Enable necessary extensions
create extension if not exists "uuid-ossp";

-- Create enum types
create type difficulty_level as enum ('easy', 'medium', 'hard');
create type strength_level as enum ('light', 'medium', 'strong', 'none');
create type subscription_tier as enum ('free', 'premium', 'enterprise');
```

#### User Profiles (`migrations/20240101000002_profiles_table.sql`)
```sql
-- User profiles table
create table public.profiles (
  id uuid references auth.users on delete cascade primary key,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null,
  updated_at timestamp with time zone default timezone('utc'::text, now()) not null,
  username text unique,
  full_name text,
  avatar_url text,
  my_bar_inventory jsonb default '[]'::jsonb,
  default_preferences jsonb default '{}'::jsonb,
  subscription_tier subscription_tier default 'free'::subscription_tier,
  usage_count integer default 0,
  last_generation_at timestamp with time zone
);

-- Enable RLS
alter table public.profiles enable row level security;

-- Profiles policies
create policy "Public profiles are viewable by everyone" on public.profiles
  for select using (true);

create policy "Users can insert their own profile" on public.profiles
  for insert with check (auth.uid() = id);

create policy "Users can update their own profile" on public.profiles
  for update using (auth.uid() = id);
```

#### Recipe Tables (`migrations/20240101000003_recipes_tables.sql`)
```sql
-- Saved recipes table
create table public.saved_recipes (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references public.profiles(id) on delete cascade not null,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null,
  updated_at timestamp with time zone default timezone('utc'::text, now()) not null,
  name text not null,
  recipe_data jsonb not null,
  is_favorite boolean default false,
  tags text[] default array[]::text[],
  notes text,
  rating integer check (rating >= 1 and rating <= 5)
);

-- Recipe generation history
create table public.generation_history (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references public.profiles(id) on delete cascade not null,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null,
  preferences_input jsonb not null,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kingkwongsta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
