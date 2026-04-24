---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **⚠️ MAINTENANCE GUIDELINES FOR THIS FILE**
>
> This file is an **index**, not a tutorial. When updating CLAUDE.md:
> - **DO NOT add code samples** (SQL, TypeScript, bash) - put them in `docs/` files instead
> - **DO** add brief descriptions of features with doc references
> - **DO** keep essential bash commands for development (npm scripts, docker commands)
> - **DO** keep critical warnings that prevent common mistakes (e.g., FK index requirement)
> - **EXCEPTION**: Angular patterns (Signals, OnPush) stay here as they're project-wide conventions
>
> Pattern to follow: `**Feature Name** (version): Brief description. See \`docs/path/FILE.md\` for details.`

## Project Overview

Civic OS is a meta-application framework that automatically generates CRUD (Create, Read, Update, Delete) views for any PostgreSQL database schema. The Angular frontend dynamically creates list, detail, create, and edit pages based on database metadata stored in custom PostgreSQL views.

**Key Concept**: Instead of manually building UI for each table, Civic OS reads database schema metadata from `schema_entities` and `schema_properties` views to automatically generate forms, tables, and validation.

**License**: This project is licensed under the GNU Affero General Public License v3.0 or later (AGPL-3.0-or-later). Copyright (C) 2023-2026 Civic OS, L3C. See the LICENSE file for full terms.

## Architecture

### Core Data Flow
1. **Database Schema** → PostgreSQL metadata tables (`metadata.entities`, `metadata.properties`)
2. **Metadata Views** → `schema_entities` and `schema_properties` views aggregate database structure
3. **SchemaService** → Fetches metadata and determines property types (text, number, foreign key, etc.)
4. **DataService** → Performs CRUD operations via PostgREST API
5. **Dynamic Pages** → List/Detail/Create/Edit pages render based on schema metadata
6. **Smart Components** → `DisplayPropertyComponent` and `EditPropertyComponent` adapt to property types

### Key Services

**SchemaService** (`src/app/services/schema.service.ts`) - Fetches and caches entity and property metadata, determines property types from PostgreSQL data types (e.g., `int4` with `join_column` → `ForeignKeyName`), filters properties for different contexts (list, detail, create, edit). Uses hybrid signal + observable pattern with in-flight request tracking to prevent duplicate HTTP requests (83% reduction in network traffic). See `docs/development/SCHEMA_SERVICE_ARCHITECTURE.md` for implementation details

**DataService** (`src/app/services/data.service.ts`) - Abstracts PostgREST API calls, builds query strings with select fields, ordering, and filters

**AuthService** (`src/app/services/auth.service.ts`) - Integrates with Keycloak for authentication via `keycloak-angular` library

### Property Type System

The `EntityPropertyType` enum maps PostgreSQL types to UI components:
- `ForeignKeyName`: Integer/UUID with `join_column` → Dropdown with related entity's display_name
- `User`: UUID with `join_table = 'civic_os_users'` → User display component with unified view access. See `docs/development/PROPERTY_TYPE_REFERENCE.md` for architecture details.
- `Payment`: UUID FK to `payments.transactions` → Payment status badge display, "Pay Now" button on detail pages (v0.13.0+)
- `DateTime`, `DateTimeLocal`, `Date`: Timestamp types → Date/time inputs
- `Boolean`: `bool` → Checkbox
- `Money`: `money` → Currency input (ngx-currency)
- `IntegerNumber`: `int4`/`int8` → Number input
- `TextShort`: `varchar` → Text input
- `TextLong`: `text` → Textarea
- `GeoPoint`: `geography(Point, 4326)` → Interactive map (Leaflet) with location picker
- `Color`: `hex_color` → Color chip display with native HTML5 color picker
- `Email`: `email_address` → Clickable mailto: link, HTML5 email input
- `Telephone`: `phone_number` → Clickable tel: link with formatted display, masked input (XXX) XXX-XXXX
- `TimeSlot`: `time_slot` (tstzrange) → Formatted date range display, dual datetime-local inputs with validation, optional calendar visualization

**Calendar Integration** (v0.9.0+): Enable via `show_calendar=true` and `calendar_property_name` in `metadata.entities`. Supports overlap prevention via GIST exclusion constraints. See `docs/development/CALENDAR_INTEGRATION.md` for details and `examples/community-center/` for working example.

**iCal Subscription Feeds** (v0.27.0+): Export public calendar events as subscribable iCal feeds. Uses RFC 5545 helper functions (`format_ical_event()`, `wrap_ical_feed()`) with PostgREST media type handlers. Calendar apps (Google Calendar, Apple, Outlook) can subscribe to `/rpc/your_feed_function`. See `docs/INTEGRATOR_GUIDE.md` (iCal Calendar Feeds section) for implementation guide.

**Status Type** (`Status`, v0.15.0+): Centralized workflow system using `metadata.statuses` table with `entity_type` discriminator. Features colored badges/dropdowns, `is_initial`/`is_terminal` states, allowed transitions (v0.33.0+), and `status_key` for programmatic references. See `docs/INTEGRATOR_GUIDE.md` (Status Type System section) and `docs/development/STATUS_TYPE_SYSTEM.md` for design.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/civic-os) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
