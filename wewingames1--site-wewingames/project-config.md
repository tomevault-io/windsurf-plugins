---
trigger: always_on
description: WeWinGames is a comprehensive sports betting information and picks service built with Laravel 12 and Vue.js 3. The platform provides betting recommendations, game analysis, subscription-based access to premium picks, and a full suite of content management and user engagement features.
---

# WeWinGames - Sports Betting Platform

## Overview
WeWinGames is a comprehensive sports betting information and picks service built with Laravel 12 and Vue.js 3. The platform provides betting recommendations, game analysis, subscription-based access to premium picks, and a full suite of content management and user engagement features.

## Technology Stack

### Backend
- **Framework**: Laravel 12 (PHP 8.2+)
- **Database**: MySQL 8.0 / SQLite (local development)
- **Cache**: File/Redis with Cloudflare integration
- **Queue**: Laravel Queue with database driver
- **Authentication**: Laravel Breeze with Inertia.js
- **Billing**: Laravel Cashier (Stripe integration)
- **SSR**: Inertia.js v2

### Frontend
- **Framework**: Vue.js 3 with TypeScript
- **Build Tool**: Vite 6
- **CSS**: Bootstrap 5 (migrated from Tailwind CSS)
- **UI Components**: Bootstrap 5 components with custom admin theme
- **Rich Text**: TinyMCE and Tiptap editors
- **Charts**: Chart.js with vue-chartjs
- **3D Graphics**: Three.js
- **Icons**: Bootstrap Icons, Lucide Vue, Heroicons

### Third-Party Services
- **Payment**: Stripe (with dynamic product management)
- **Push Notifications**: OneSignal & Web Push API
- **Analytics**: Google Analytics & Tag Manager
- **Security**: Cloudflare Turnstile
- **Email**: SendGrid (via LoggedMailChannel)
- **Marketing**: SpringBig (member sync & tier segments)
- **Monitoring**: Laravel Telescope
- **Media**: Spatie Media Library
- **Permissions**: Spatie Laravel Permission
- **Activity Logging**: Spatie Activity Log

## Project Structure

```
.
├── app/
│   ├── Console/           # Artisan commands
│   ├── Events/            # Event classes
│   ├── Http/
│   │   ├── Controllers/   # All controllers including Admin/
│   │   ├── Middleware/    # Custom middleware
│   │   └── Requests/      # Form requests
│   ├── Models/            # Eloquent models (30+ models)
│   ├── Services/          # Business logic services
│   ├── Policies/          # Authorization policies
│   └── Mail/              # Mailable classes
├── resources/
│   ├── js/
│   │   ├── components/    # Reusable Vue components
│   │   ├── pages/         # Page components
│   │   ├── layouts/       # Layout components
│   │   └── composables/   # Vue composition utilities
│   └── css/               # Bootstrap-based styles
├── routes/                # Application routes
├── database/              # Migrations, seeders, factories
├── tests/                 # PHPUnit & Feature tests
└── docker/                # Docker configuration
```

## Key Features

### 1. Betting System
- **Bet Management**: Create, edit, track betting picks with performance metrics
- **Parlay Support**: Multi-bet parlays with combined odds
- **Golf Betting**: Each-way bets with place fractions and dead heat rules
- **CSV Import/Export**: Wizard-based bulk data management
- **Mass Edit**: Batch updates for golf positions
- **Premium Notes**: Subscriber-only betting insights
- **Profit Tracking**: Detailed P&L calculations

### 2. User & Subscription System
- **User Types**: Regular, Ambassador, Gifted, Admin
- **Subscription Tiers**: Bronze, Silver, Gold, Platinum
- **Billing Periods**: Daily, Weekly, Monthly, Yearly
- **Dynamic Stripe Products**: Database-driven product management
- **Discount Codes**: Percentage/fixed with usage limits
- **Affiliate System**: Track and manage affiliates
- **Impersonation**: Admin user switching
- **Quick Checkout**: Payment-first registration flow (feature flagged)

### 3. Content Management
- **Blog System**: Full-featured with SEO, categories, view tracking
- **CMS Pages**: Dynamic page creation and management
- **Landing Pages**: Marketing-focused pages
- **FAQ System**: Categorized Q&A management
- **Knowledgebase**: Article-based help system
- **Media Library**: Centralized file management
- **Testimonials**: Customer reviews with Google integration

### 4. Communication Features
- **Email System**: 
  - Template management
  - Full logging with SendGrid
  - Customizable transactional emails
- **Push Notifications**:
  - OneSignal integration (NEW)
  - Web Push API fallback
  - Tier-based targeting
  - Notification history
- **Support Tickets**: Guest-accessible support system

### 5. Career/Jobs System
- **Job Positions**: Manage job listings
- **Resume Submissions**: Application tracking system
- **Admin Review**: Application management interface

### 6. Admin Dashboard
Located at `/admin`, provides:
- **Statistics Dashboard**: MRR, user growth, betting activity
- **User Management**: Complete user administration
- **Bet Management**: Full CRUD with import/export
- **Content Editing**: Pages, posts, FAQs, testimonials
- **Subscription Dashboard**: Customer & revenue tracking
- **System Settings**: Configuration management
- **Activity Logs**: User action tracking
- **Cache Management**: Clear Laravel & Cloudflare cache

### 7. Security & Performance
- **Middleware**:
  - Admin security headers
  - Rate limiting
  - IP blacklisting
  - Spam prevention
- **Under Construction Mode**: Site-wide maintenance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WeWinGames1/SITE-WeWinGames](https://github.com/WeWinGames1/SITE-WeWinGames) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
