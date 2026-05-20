---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Warolabs.com is a Nuxt.js application focused on democratizing technology and AI knowledge through virtual events, workshops, and educational content. The platform serves Spanish-speaking audiences and includes lead capture, email marketing campaigns, and user management functionality.

## Development Commands

```bash
# Development server (runs on port 4000)
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Database operations
npm run db:generate    # Generate Drizzle migrations
npm run db:migrate     # Run database migrations

# Testing
npm run test          # Run Jest tests

# Check database connection
npm run check-db-url  # Verify DATABASE_URL environment variable
```

## Architecture

### Tech Stack
- **Frontend**: Nuxt 3 (Vue.js) with Server-Side Rendering
- **Backend**: Nuxt API routes with H3 event handlers
- **Database**: PostgreSQL with Drizzle ORM
- **Authentication**: Better Auth with magic link authentication
- **Email**: AWS SES for transactional emails
- **Styling**: TailwindCSS with SCSS

### Database Schema
The application uses a comprehensive multi-tenant architecture. For detailed schema information, refer to `BASE_DATA_WAROLABS.txt` in the project root (gitignored).

#### Core Entities:
**User Management:**
- `profile` - Main user profiles with contact info, company details, and personal information
- `sessions` - User sessions for Better Auth
- `magic_tokens` - Passwordless authentication tokens
- `addresses` - User addresses (billing/shipping)

**Multi-tenancy:**
- `tenants` - Organizations/workspaces with slug-based routing
- `tenant_members` - User-tenant relationships with role-based access
- `tenant_invitations` - Pending tenant invitations with email tokens

**Content Management:**
- `articles` - Blog posts/articles with SEO metadata, multilingual support
- `images` - File storage metadata with MIME types and descriptions
- `profile_images` - User profile image associations
- `comments` - Comment system with emotional reactions and moderation
- `commentable_items` - Polymorphic comment associations

**E-commerce/Events:**
- `clusters` - Event/venue containers with legal info and settings
- `areas` - Event sections/areas with pricing and capacity
- `units` - Individual bookable units within areas
- `reservations` - Booking records with date ranges and status
- `reservation_units` - Junction table for unit reservations with transfer capabilities
- `cancellations` - Cancellation records with refund status

**Product Catalog:**
- `product` - Product definitions with pricing
- `product_drop` - Product collections/releases
- `product_variants` - SKU variations with attributes and inventory
- `product_images` - Product image associations
- `categories` - Product categorization

**Order Management:**
- `orders` - Purchase orders with tracking and analytics data
- `order_items` - Line items with pricing snapshots
- `order_status_history` - Order state change tracking
- `payments` - Payment processing with gateway integration
- `inventory_transactions` - Stock movement tracking

**Marketing & CRM:**
- `campaign` - Marketing campaign management
- `leads` - Lead capture with UTM tracking and analytics
- `campaign_leads` - Campaign-lead associations
- `templates` - Email template system
- `template_versions` - Template versioning
- `campaign_template_versions` - Campaign-template associations
- `email_opens` - Email open tracking for campaigns with IP and user agent data
- `email_clicks` - Email click tracking for campaigns with original URLs and analytics
- `lead_interactions` - Unified tracking of all lead behavior and journey stages
- `lead_groups` - Smart lead segmentation based on behavior and engagement
- `lead_group_members` - Lead-group associations for targeted campaigns

**Pricing & Promotions:**
- `promotions` - Discount system targeting various entities
- `sale_stages` - Time-based pricing tiers
- `legal_info` - Company legal information for events

**Misc:**
- `unit_transfer_log` - Unit ownership transfer history
- `cluster_images` - Event/venue image associations
- `reservation_unit_qr_images` - QR code images for reservations

### API Structure
API routes are organized under `server/api/` with the following pattern:
- `/api/auth/` - Authentication endpoints (magic links, tokens)
- `/api/campaign/` - Email campaign management
- `/api/marketing/` - Marketing and lead capture
- `/api/profiles/` - User profile management
- `/api/landings/` - Landing page data

### Key Utilities
- `server/utils/basedataSettings/withPostgresClient.js` - Database connection wrapper
- `server/utils/aws/sesClient.js` - AWS SES email sending
- `server/utils/security/` - JWT verification and authorization
- `server/utils/emailTemplates/` - Email template rendering

## Configuration

### Database Schema Reference
The complete database schema dump is available in `BASE_DATA_WAROLABS.txt` in the project root. This file contains:
- Complete table definitions with all columns and data types
- Primary key, foreign key, and unique constraints
- Indexes for performance optimization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uno0uno/warolabs.com](https://github.com/uno0uno/warolabs.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
