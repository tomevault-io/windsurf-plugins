---
trigger: always_on
description: A web-based event/activity calendar for the Department of Trade and Industry (DTI) Caraga regional office. It manages events organized by office → division → unit hierarchy, with geographic tagging (region → province → LGU → barangay), and role-based access control.
---

# DTI Caraga Calendar of Events System

A web-based event/activity calendar for the Department of Trade and Industry (DTI) Caraga regional office. It manages events organized by office → division → unit hierarchy, with geographic tagging (region → province → LGU → barangay), and role-based access control.

---

## Tech Stack

### Backend
| Component | Version |
|-----------|---------|
| Python | 3.12 |
| Django | 5.2 (installed), settings.py comment says 4.2.4 — ignore the comment |
| PostgreSQL | Default port 5432, db name: `db_appcal` |
| psycopg2-binary | 2.9.11 |
| django-bootstrap4 | 26.1 |

### Frontend (all via local static files, no npm/build step)
| Component | Version |
|-----------|---------|
| Vue.js 3 | CDN global build (`vue.global.js`) |
| Bootstrap | 5.3.2 |
| jQuery | 3.7.0 |
| jQuery UI | Included (datepicker) |
| jQuery DataTables | Included (server-side processing) |

### Key Python packages (no requirements.txt — use `pip freeze`)
- `asgiref` 3.8.1
- `sqlparse` 0.5.3
- `psycopg2-binary` 2.9.11
- `django-bootstrap4` 26.1

---

## Dev Commands

```bash
# Run dev server
python manage.py runserver

# Run on local network (accessible from other machines)
python manage.py runserver 0.0.0.0:8000

# Apply migrations
python manage.py migrate

# Create new migrations after model changes
python manage.py makemigrations

# Create superuser
python manage.py createsuperuser

# Django admin
# http://localhost:8000/admin/

# Dump/load data
python manage.py dumpdata
python manage.py loaddata

# No test suite configured — no test runner commands currently in use
```

---

## Directory Structure

```
dticalendar/
├── dticalendar/          # Django project config
│   ├── settings.py       # All config: DB, email, apps, static/media paths
│   ├── urls.py           # Root URL dispatcher
│   └── wsgi.py           # WSGI entry point
│
├── users/                # Auth, registration, profile, user management
├── events/               # Core Event model and CRUD (most complex app)
├── calendars/            # Calendar grouping for events
├── divisions/            # Division management (belongs to Office)
├── units/                # Unit management (belongs to Division)
├── offices/              # Office management (top-level org unit)
├── employees/            # Employee records (minimal, mostly unused)
├── orgoutcomes/          # Organizational Outcomes (linked to PAPs)
├── paps/                 # Priority Action Plans (linked to OrgOutcomes)
├── regions/              # Region reference data (read-only)
├── provinces/            # Province reference data
├── lgus/                 # Local Government Units
├── barangays/            # Barangay reference data
│
├── static/               # All static assets (JS, CSS — no build step)
│   ├── js/
│   │   ├── custom-vue.js          # Main Vue app: form modals, CRUD, dropdowns
│   │   ├── custom-events-vue.js   # Events display Vue app: DataTables, pills
│   │   ├── vue.global.js          # Vue 3 runtime
│   │   ├── bootstrap5.3.2.bundle.min.js
│   │   ├── jquery-3.7.0.js
│   │   ├── jquery.dataTables.min.js
│   │   └── [jszip, pdfmake, popper, etc.]
│   └── css/
│
├── templates/            # Global templates directory
│   ├── users/
│   │   ├── master.html   # Base template — all JS/CSS loaded here
│   │   ├── profile.html  # Main dashboard (logged-in home page)
│   │   ├── register.html
│   │   ├── login.html
│   │   └── manage_users.html
│   └── registration/     # Django password reset templates
│
├── events/templates/events/
│   ├── event_display.html        # All-events DataTable view
│   ├── events_display_div.html   # Events filtered by division
│   └── events_display_unit.html  # Events filtered by unit
│
└── media/                # User-uploaded file attachments (FileField uploads)
```

---

## Architecture Patterns

### Request/Response Pattern
All CRUD operations are done via AJAX — no traditional form POSTs that redirect. The pattern is:

```python
# views.py
@login_required
@csrf_exempt
def save_something_ajax(request):
    if request.method == 'POST':
        # validate & save
        return JsonResponse({'message': 'True'})
    return JsonResponse({'message': 'False'})
```

Frontend checks `response.message === 'True'` and reloads/updates accordingly.

### Permission Levels
Three tiers, checked inline in each view (no Django permissions framework):

1. **Superuser** — full access to everything
2. **Office Admin** (`UserProfile.is_office_admin = True`) — can manage Division, Unit, OrgOutcome, PAP for their assigned office
3. **Regular User** — can only create/edit events via the profile page

Each app that needs it defines its own `_can_manage(user, office_id)` or `_is_authorized(user)` helper at the top of `views.py`.

### DataTables Server-Side Processing
All list views (Division, Unit, Office, OrgOutcome, PAP, Users) use jQuery DataTables with server-side processing. Views handle `draw`, `start`, `length`, `search[value]`, `order[0][column]`, `order[0][dir]` query params and return `{draw, recordsTotal, recordsFiltered, data}`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/j-curato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
