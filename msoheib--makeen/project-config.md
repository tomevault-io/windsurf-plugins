---
trigger: always_on
description: - **Supabase Project ID**: `fbabpaorcvatejkrelrf`
---

# Real Estate Management Database Schema Documentation

## Project Information
- **Supabase Project ID**: `fbabpaorcvatejkrelrf`
- **Project Name**: `property-management` 
- **Region**: `eu-central-1`
- **Database Version**: PostgreSQL 15.8.1.093
- **Status**: ACTIVE_HEALTHY
- **Host**: `db.fbabpaorcvatejkrelrf.supabase.co`

## Schema Overview

The database is designed for a comprehensive real estate management system supporting property management, tenant/owner relationships, financial transactions, maintenance tracking, document management, and accounting functions.

### Core Business Entities
1. **People Management** - `profiles`, `clients`
2. **Property Management** - `properties`, `property_reservations`
3. **Contract Management** - `contracts` 
4. **Maintenance Operations** - `maintenance_requests`, `work_orders`
5. **Financial Management** - `vouchers`, `invoices`, `accounts`, `cost_centers`
6. **Communications** - `letters`, `issues`
7. **Document Management** - `documents`
8. **Asset Management** - `fixed_assets`

---

## Table Definitions

### 1. PROFILES
**Purpose**: Central table for all people in the system (users, tenants, owners, employees, contractors)

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `first_name` | text | YES | NULL | |
| `last_name` | text | YES | NULL | |
| `email` | text | YES | NULL | Unique |
| `role` | text | YES | `'tenant'` | CHECK: admin, manager, owner, tenant, buyer, employee, contractor |
| `phone` | text | YES | NULL | |
| `address` | text | YES | NULL | |
| `city` | text | YES | NULL | |
| `country` | text | YES | NULL | |
| `nationality` | text | YES | NULL | |
| `id_number` | text | YES | NULL | |
| `is_foreign` | boolean | YES | `false` | |
| `profile_type` | text | YES | NULL | CHECK: owner, tenant, buyer, employee, admin |
| `status` | text | YES | `'active'` | CHECK: active, inactive, suspended |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Referenced By
- `properties.owner_id` → `profiles.id`
- `contracts.tenant_id` → `profiles.id`
- `maintenance_requests.tenant_id` → `profiles.id`
- `work_orders.assigned_to` → `profiles.id`
- `vouchers.tenant_id` → `profiles.id`
- `vouchers.created_by` → `profiles.id`
- `invoices.tenant_id` → `profiles.id`
- `letters.sender_id` → `profiles.id`
- `issues.reported_by` → `profiles.id`
- `documents.uploaded_by` → `profiles.id`

---

### 2. PROPERTIES
**Purpose**: Property listings and management information

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `title` | text | NO | NULL | |
| `description` | text | YES | NULL | |
| `property_type` | text | NO | NULL | CHECK: apartment, villa, office, retail, warehouse |
| `status` | text | YES | `'available'` | CHECK: available, rented, maintenance, reserved |
| `address` | text | NO | NULL | |
| `city` | text | NO | NULL | |
| `country` | text | NO | NULL | |
| `neighborhood` | text | YES | NULL | |
| `area_sqm` | numeric | NO | NULL | |
| `bedrooms` | integer | YES | NULL | |
| `bathrooms` | integer | YES | NULL | |
| `price` | numeric | NO | NULL | |
| `payment_method` | text | YES | `'cash'` | CHECK: cash, installment |
| `owner_id` | uuid | YES | NULL | FK to profiles.id |
| `images` | text[] | YES | NULL | |
| `property_code` | text | YES | NULL | |
| `floor_number` | integer | YES | NULL | |
| `building_name` | text | YES | NULL | |
| `parking_spaces` | integer | YES | `0` | |
| `amenities` | text[] | YES | NULL | |
| `is_furnished` | boolean | YES | `false` | |
| `annual_rent` | numeric | YES | NULL | |
| `service_charge` | numeric | YES | `0` | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `owner_id` → `profiles.id` (ON DELETE RESTRICT)

#### Referenced By
- `contracts.property_id` → `properties.id`
- `maintenance_requests.property_id` → `properties.id`
- `vouchers.property_id` → `properties.id`
- `invoices.property_id` → `properties.id`
- `property_reservations.property_id` → `properties.id`
- `issues.property_id` → `properties.id`
- `fixed_assets.property_id` → `properties.id`

---

### 3. CONTRACTS
**Purpose**: Rental, sales, and management contracts

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `property_id` | uuid | YES | NULL | FK to properties.id |
| `tenant_id` | uuid | YES | NULL | FK to profiles.id |
| `start_date` | timestamptz | NO | NULL | |
| `end_date` | timestamptz | NO | NULL | |
| `rent_amount` | numeric | NO | NULL | |
| `payment_frequency` | text | YES | `'monthly'` | CHECK: monthly, quarterly, biannually, annually |
| `security_deposit` | numeric | NO | NULL | |
| `is_foreign_tenant` | boolean | YES | `false` | |
| `status` | text | YES | `'active'` | CHECK: active, expired, terminated, renewal |
| `documents` | text[] | YES | NULL | |
| `contract_number` | text | YES | NULL | |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/msoheib) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
