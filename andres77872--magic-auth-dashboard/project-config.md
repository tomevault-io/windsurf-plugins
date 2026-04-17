---
trigger: always_on
description: Below is a concise, human-readable reference of every REST endpoint defined in the supplied OpenAPI document.
---

Below is a concise, human-readable reference of every REST endpoint defined in the supplied OpenAPI document.  
For each route you’ll find the HTTP method, a short description (summary + key points from the longer description) and the request parameters (path, query, and/or body-form fields).

> • “(req)” = required • Location is shown in parentheses → path | query | form-body  
> • When a route accepts both Bearer-header and secure HTTP-only cookies, only the business parameters are listed.

---

# 3-Tier User-Type Multi-Project Auth API – Endpoint Reference

## 1. Authentication
### POST  /auth/login
Authenticate user and start a session (sets HTTP-only cookie).  
Params  
• username (form, req) • password (form, req) • project_hash (form, optional – root may omit)

### POST  /auth/register
Register a new user and auto-assign default group.  
Params  
• username (form, req) • password (form, req) • email (form, optional) • project_hash (form, req)

### GET  /auth/validate
Validate current session token and return user & group context.  
Params – none

### POST  /auth/logout
Invalidate the current session & clear cookie.  
Params – none

### POST  /auth/refresh
Refresh JWT, extending the session.  
Params – none

### POST  /auth/switch-project
Switch active project in an existing session.  
Params  
• project_hash (form, req)

### POST  /auth/check-availability
Check if a username or e-mail is free globally.  
Params  
• username (form, optional) • email (form, optional)

---

## 2. User Management
### GET  /users/profile
Return current user profile, groups & accessible projects.  
Params – none

### PUT  /users/profile
Update own username / email / password.  
Params  
• username (form, optional) • email (form, optional) • password (form, optional)

### GET  /users/access-summary
Short access summary (groups, projects, permissions).  
Params – none

### GET  /users
List users with filters (admin/root only).  
Query params  
• limit (default 50) • offset (default 0) • user_type • search • is_active

### GET  /users/{user_hash}
Get details for a specific user (self or admin/root scope).  
Path param • user_hash

### PATCH  /users/{user_hash}/status
Activate / deactivate a user (admin).  
Path • user_hash Form • is_active (req)

### POST  /users/{user_hash}/reset-password
Admin reset => returns temporary password.  
Path • user_hash

### PATCH  /users/{user_hash}/type
ROOT-only change of user-type (root, admin, consumer).  
Path • user_hash Form • user_type (req)

---

## 3. User-Type Management
### POST  /user-types/root
Create a new root (super-admin).  
Form • username (req) • password (req) • email (optional)

### POST  /user-types/admin
Create an admin for one / many projects.  
Form • username (req) • password (req) • email (req) • assigned_project_id OR assigned_project_ids

### GET  /user-types/{user_hash}/info
Comprehensive type info for a user.  
Path • user_hash

### PUT  /user-types/{user_hash}/type
ROOT-only promotion/demotion + project assignment.  
Path • user_hash Form • user_type (req) • assigned_project_id (optional)

### GET  /user-types/users/{user_type}
List all users of a given type.  
Path • user_type Query • limit • offset

### PUT  /user-types/admin/{user_hash}/projects
Overwrite the list of projects an admin manages.  
Path • user_hash Form • assigned_project_ids (req)

### GET  /user-types/admin/{user_hash}/projects
Fetch all projects an admin currently manages.  
Path • user_hash

### POST  /user-types/admin/{user_hash}/projects/add
Add an extra project to an admin.  
Path • user_hash Form • project_id (req)

### DELETE  /user-types/admin/{user_hash}/projects/{project_id}
Remove admin from a specific project.  
Path • user_hash • project_id

### GET  /user-types/stats
System-wide distribution of user types.  
Params – none

---

## 4. Project Management
### GET  /projects
List projects visible to caller.  
Query • limit (1-100, default 10) • offset • search

### POST  /projects
Create a new project (admin/root).  
Form • project_name (req) • project_description (optional)

### GET  /projects/{project_hash}
Detailed project info with caller’s access context.  
Path • project_hash

### PUT  /projects/{project_hash}
Update project metadata (admin).  
Path • project_hash Form • project_name • project_description

### DELETE  /projects/{project_hash}
Delete / revoke access to a project (admin).  
Path • project_hash

### GET  /projects/{project_hash}/members
List members of a project.  
Path • project_hash Query • limit • offset • user_type

### POST  /projects/{project_hash}/members
Add a member to a project with role.  
Path • project_hash Form • user_hash (req) • role (default consumer)

### DELETE  /projects/{project_hash}/members/{user_hash}
Remove member from project.  
Path • project_hash • user_hash

### GET  /projects/{project_hash}/activity
Project activity feed (phase 2).  
Path • project_hash Query • limit • offset • activity_type • days

### GET  /projects/{project_hash}/stats
Detailed stats (phase 2).  
Path • project_hash

### PATCH  /projects/{project_hash}/owner
Transfer ownership (admin).  
Path • project_hash Form • new_owner_hash (req)

### PATCH  /projects/{project_hash}/archive

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Andres77872) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
