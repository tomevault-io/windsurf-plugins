---
trigger: always_on
description: Проект представляет собой SaaS-систему для автоматического составления университетских расписаний. Основа - существующее Laravel-приложение с MVC-архитектурой, Filament-админкой и Docker-контейнеризацией. Цель - превращение в production-ready SaaS-продукт с автогенерацией расписаний и мульти-арендой.
---

# SPEC-1: University SaaS Scheduling System

## Overview

Проект представляет собой SaaS-систему для автоматического составления университетских расписаний. Основа - существующее Laravel-приложение с MVC-архитектурой, Filament-админкой и Docker-контейнеризацией. Цель - превращение в production-ready SaaS-продукт с автогенерацией расписаний и мульти-арендой.

## Architecture

### Tech Stack

- **Backend**: Laravel 10 (PHP 8.2)
- **Frontend**: Filament Admin (Alpine.js + Tailwind)
- **Database**: MySQL (может быть заменён на PostgreSQL)
- **Queue**: Redis
- **Solver Service**: Go с CP-SAT алгоритмом
- **Containerization**: Docker Compose

### Component Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        NGINX                                 │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                   Laravel App                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Models    │  │  Filament  │  │   API Controllers   │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│         │                │                    │              │
│  ┌──────▼────────────────▼────────────────────▼──────────┐  │
│  │              Tenant Manager (Middleware)               │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────┬───────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
┌───────▼──────┐ ┌────▼────┐ ┌────▼────────┐
│   MySQL      │ │  Redis  │ │  Go Solver  │
│  (tenants)   │ │ (queue) │ │   Service   │
└──────────────┘ └─────────┘ └─────────────┘
```

## Database Schema

### Tenant Isolation

Все таблицы содержат `tenant_id` (UUID) для изоляции данных между арендаторами.

### Core Tables

| Table | Description |
|-------|-------------|
| `tenants` | Арендаторы (вузы/организации) |
| `users` | Пользователи с ролями (owner/admin/planner/teacher/viewer) |
| `rooms` | Аудитории с вместимостью и типами |
| `calendars` | Календари (семестры) с настройками |
| `time_slots` | Временные слоты (день/номер пары/время) |
| `activities` | Активности (занятия для размещения) |
| `activity_groups` | Связь активностей с группами |
| `activity_teachers` | Связь активностей с преподавателями |
| `teacher_unavailability` | Недоступность преподавателей |
| `teacher_preferences` | Пожелания преподавателей |
| `soft_weights` | Веса мягких ограничений |
| `schedule_versions` | Версии расписаний (draft/published/archived) |
| `schedule_assignments` | Назначения занятий в слоты |
| `violations` | Нарушения ограничений |
| `audit_logs` | Логи аудита |
| `import_jobs` | Задания импорта |

## Models

### Tenant
```php
App\Models\Tenant
- id (uuid, primary)
- name, subdomain, domain
- settings (jsonb) - настройки арендатора
- relationships: users, rooms, calendars, activities, scheduleVersions
```

### Room
```php
App\Models\Room
- id, tenant_id (uuid)
- code, title, capacity
- room_type (lecture/lab/seminar/pc/gym/other)
- features (jsonb), active (bool)
```

### Calendar
```php
App\Models\Calendar
- id, tenant_id (uuid)
- name, start_date, end_date
- weeks, parity_enabled
- days_per_week, slots_per_day
- slot_duration_minutes, break_duration_minutes
```

### Activity
```php
App\Models\Activity
- id, tenant_id (uuid)
- subject_id, calendar_id
- title, activity_type, duration_slots
- required_slots_per_period
- relationships: subject, calendar, groups, teachers
```

### ScheduleVersion
```php
App\Models\ScheduleVersion
- id, tenant_id (uuid)
- calendar_id, name, status (draft/published/archived)
- created_by, parent_version_id
- version_number, random_seed (для воспроизводимости)
- generation_params (jsonb), published_at
```

## Multi-Tenancy Implementation

### TenantManager

Сервис для управления текущим арендатором:

```php
App\Services\TenantManager
- getTenant(): ?Tenant
- setTenant(Tenant $tenant): void
- setTenantById(string $tenantId): ?Tenant
- resolveTenantId(): ?string (из subdomain/domain/auth)
- clearTenant(): void
```

### TenantMiddleware

Middleware для определения арендатора из HTTP-запроса:
1. Проверяет subdomain/domain в URL
2. Проверяет tenant_id авторизованного пользователя
3. Устанавливает tenant в контейнер приложения

### TenantScope Trait

Трейт для автоматической фильтрации по tenant_id:

```php
App\Models\Traits\TenantScope
- scopeTenant(Builder $query, ?string $tenantId = null): Builder
- getTenantId(): ?string
- bootTenantScope(): auto-set tenant_id при создании
```

## Filament Admin Resources

### Available Resources

| Resource | Navigation Group | Description |
|----------|-----------------|-------------|
| `TenantResource` | SaaS | Управление арендаторами |
| `RoomResource` | Розклад | Аудитории |
| `CalendarResource` | Розклад | Календари/семестры |
| `ActivityResource` | Розклад | Активности/занятия |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sviniabanditka/scheduler](https://github.com/sviniabanditka/scheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
