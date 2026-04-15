---
trigger: always_on
description: This is a small **Laravel 12** web app for managing **leave and mission requests**. Users can submit requests; certain users (role `approver`) can approve or reject them. A **department administrator** (role `admin`) can also view and submit requests on behalf of other users.
---

# Copilot Instructions for WorkflowSystem

## 📌 What is this repo?

This is a small **Laravel 12** web app for managing **leave and mission requests**. Users can submit requests; certain users (role `approver`) can approve or reject them. A **department administrator** (role `admin`) can also view and submit requests on behalf of other users.

## 🚀 How to run locally

### 1) Install dependencies

- `composer install`
- `npm install`

### 2) Setup environment

- Copy `.env.example` to `.env` (if missing)
- Set `APP_URL=http://localhost:8000` (default)

### 3) Migrate database

- `php artisan migrate`

### 4) Run the app

- `php artisan serve`
- Visit `http://localhost:8000`

### 5) Test

- `php artisan test`

## 🧠 Architecture & key places

- **HTTP routes**: `routes/web.php`
- **Controllers**:
    - `app/Http/Controllers/LeaveRequestController.php`
    - `app/Http/Controllers/MissionRequestController.php`
    - `app/Http/Controllers/ApproverController.php`
    - `app/Http/Controllers/LoginController.php`
- **Models**:
    - `app/Models/LeaveRequest.php`
    - `app/Models/MissionRequest.php`
    - `app/Models/User.php` (role logic lives here via `isApprover()`)
- **Views**:
    - `resources/views/dashboard.blade.php` (user dashboard)
    - `resources/views/approver/dashboard.blade.php` (approver dashboard)
    - `resources/views/leave_requests/*` and `resources/views/mission_requests/*`

## ✅ Project-Specific Conventions

**User Roles & Authorization:**
- Three roles stored in `users.role` column: `'user'`, `'approver'`, `'admin'`
  - `'admin'` = Department administrator (not system admin)
  - Methods in `User` model: `isApprover()`, `isDepartmentAdmin()`, `isApproverOrDepartmentAdmin()`
- Default middleware: `EnsureUserIsApprover` (used for approver/admin routes)
- Routes protected by `auth` middleware; approver routes also require `EnsureUserIsApprover`

**Request Status Values:**
- Database enum: `'pending'`, `'approved'`, `'rejected'` (default: `'pending'`)
- All requests default to pending status on creation

**Admin/Department Admin Behavior:**
- Department admins (`admin` role) can:
  - Submit leave & mission requests on behalf of other users (via `user_id` form field)
  - Create new users (via `/users/create` route)
  - View all requests + approver dashboard
- Regular users can only see/submit their own requests

**Demo Authentication:**
- `LoginController::login()` uses `firstOrCreate()` to allow any email/password combo
- Special emails get auto-assigned roles: `admin@example.com` → `approver`, `deptadmin@example.com` → `admin`
- **This is demo-only** for testing; switch to proper auth logic when needed

**Request Submission Pattern:**
- Both `LeaveRequestController` and `MissionRequestController` follow same pattern:
  - If user is department admin, show user selection dropdown; validate `user_id` field
  - Otherwise, automatically assign to logged-in user
  - Redirect to dashboard with success message

**View Conventions:**
- Status badge colors in Blade: `bg-success` (approved) | `bg-danger` (rejected) | `bg-warning` (pending)
- Tables use Bootstrap classes: `table table-sm table-bordered`
- Dashboard shows different content based on user role via `Auth::user()->isApproverOrDepartmentAdmin()`

## 🛠 When making changes

- Prefer using Eloquent query methods (`->where(...)`, `->with(...)`).
- Blade templates use Bootstrap 5 for layout.
- Follow existing conventions for status badge classes:
    - `success` → approved
    - `danger` → rejected
    - `warning` → pending

## ⚠️ Potential Pitfalls & Environment Issues

1. **Demo Auth Creates Real Users**: `LoginController` auto-creates users on login. This is intentional for demo but can clutter the database. Before production, replace with standard Laravel Auth.

2. **Database Migrations Added Later**: Leave/mission tables were added *after* initial user table. Migrations are dated `2026_03_14_*`, so ensure `php artisan migrate` runs in order.

3. **Asset Pipeline**:
   - Vite watches CSS/JS in `resources/`, outputs to `public/build/`
   - Blade uses `@vite()` directive; views reference `asset()` for custom CSS
   - In development, `npm run dev` must be running separately from `php artisan serve`

4. **Test Database**: Tests use `:memory:` SQLite (see `phpunit.xml`). Migrations are NOT auto-run in tests—add `RefreshDatabase` trait to test classes if needed.

5. **Enum Columns**: Leave/mission status fields use MySQL/PostgreSQL enum type. If switching databases, verify enum support or use string with validation.

6. **Role Field Not In Initial Migration**: The `role` column was added via migration `2026_03_14_064938_add_role_to_users_table.php`. Ensure it runs after the base users table.

7. **No Email Notifications**: Approval/rejection is silent. Add mail/queue jobs if you need to notify users.

8. **No Pagination**: Approver dashboard and user dashboards fetch *all* pending/all requests. Add pagination for large datasets.

## 📂 Key Files & Reusable Patterns

**Authorization Pattern** (`app/Models/User.php`):
```php

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/makaranhet8-cell) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
