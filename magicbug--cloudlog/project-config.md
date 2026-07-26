---
trigger: always_on
description: Cloudlog is a self-hosted **PHP web application** for amateur radio contact logging. Built on **CodeIgniter 3** MVC framework with Bootstrap 5 frontend, HTMX for AJAX interactions, and jQuery for enhanced functionality.
---

# Cloudlog AI Coding Agent Instructions

## Project Overview
Cloudlog is a self-hosted **PHP web application** for amateur radio contact logging. Built on **CodeIgniter 3** MVC framework with Bootstrap 5 frontend, HTMX for AJAX interactions, and jQuery for enhanced functionality.

**Stack**: PHP 7.4+ (8.2 compatible), MySQL 5.7+, Apache/Nginx, CodeIgniter 3

UI: Default Bootstrap styling (no custom theme by default).

## Architecture

### MVC Structure (CodeIgniter 3)
- **Controllers** (`application/controllers/`): Extend `CI_Controller`, handle authentication via `user_model->validate_session()`
- **Models** (`application/models/`): Extend `CI_Model`, handle database operations (e.g., `Logbook_model`, `Stations`)
- **Views** (`application/views/`): PHP templates with Bootstrap 5 classes, loaded via `$this->load->view()`
- **Libraries** (`application/libraries/`): Custom classes like `Qra` (gridsquare calculations), `optionslib` (settings)

### Key Patterns
```php
// Controllers always check authentication first
$this->load->model('user_model');
if ($this->user_model->validate_session() == 0) {
    redirect('user/login');
}

// Authorization levels: authorize(2) for users, authorize(99) for admins
if (!$this->user_model->authorize(2)) {
    $this->session->set_flashdata('notice', 'You\'re not allowed to do that!');
    redirect('dashboard');
}
```

### Configuration
- **Config files**: `application/config/config.php` (app settings), `database.php` (DB credentials)
- Sample files: `config.sample.php`, `database.sample.php` - copy and customize for local setup
- Environment: Set in `index.php` - `define('ENVIRONMENT', 'development')` shows profiler
- Version: Defined in `config.php` as `$config['app_version'] = "2.4.5"`

## Frontend Integration

### HTMX Usage (Primary AJAX Method)
HTMX is the **preferred** method for dynamic content loading. Views use `hx-get`, `hx-post`, `hx-target` attributes:
```php
<!-- Auto-refreshing component -->
<div id="qso-last-table" hx-get="<?php echo site_url('/qso/component_past_contacts'); ?>" 
     hx-trigger="load, every 5s">
</div>

<!-- Form submission -->
<form hx-post="<?php echo site_url('logbooks/save_publicslug/'); ?>" 
      hx-target="#publicSlugForm">
```

### HTMX In Practice
- Previous QSOs widget:
    - View snippet (auto-refresh):
        ```php
        <div id="qso-last-table" hx-get="<?php echo site_url('/qso/component_past_contacts'); ?>" hx-trigger="load, every 5s"></div>
        ```
    - Controller endpoint: `application/controllers/Qso.php::component_past_contacts()` loads `application/views/qso/components/previous_contacts.php` with `$this->logbook_model->last_custom('5')`.

- Save Public Slug form:
    - View snippet:
        ```php
        <form hx-post="<?php echo site_url('logbooks/save_publicslug/'); ?>" hx-target="#publicSlugForm">
            <input type="hidden" name="logbook_id" value="<?php echo $logbook_id; ?>">
            <input type="text" name="public_slug" required>
            <button type="submit" class="btn btn-primary">Save</button>
        </form>
        ```
    - Controller endpoint: `application/controllers/Logbooks.php::save_publicslug()` validates `public_slug` (`required|alpha_numeric`) and persists via `logbooks_model`.

### Assets
- **Most assets**: Live under `assets/`; core includes are wired via `application/views/interface_assets/header.php` and `application/views/interface_assets/footer.php`.
- **CSS**: `assets/css/` - Bootstrap themes, custom overrides in `themes/*/overrides.css`
- **JS**: `assets/js/` - jQuery, HTMX (`htmx.min.js`), Leaflet maps, custom logic
- **Icons**: Font Awesome (via `assets/fontawesome/`)

## Database

### Migrations
Database schema managed via **CodeIgniter migrations** (`application/migrations/`):
- Sequential numbered files: `001_add_lotw_credentials.php` → `232_tag_2_7_7.php`
- Each extends `CI_Migration` with `up()` method
- Run via: `php index.php migrate` or through admin interface

### Key Tables
- `TABLE_HRD_CONTACTS_V01`: QSO log (configured in `config.php`)
- `station_profile`: Station locations and settings
- `station_logbooks`: Logbook definitions
- `station_logbooks_entity`: Logbook-location relationships

## Development Workflows

### Docker Development Setup
```bash
# Start environment (web + db services)
docker-compose up

# Access: http://localhost/
# DB host in Docker: 'db' (service name, not localhost)
```

**Configuration**: Copy `.env.sample` to `.env` and adjust DB settings before starting.

### Testing
**Cypress** end-to-end tests (`cypress/e2e/`):
```bash
# Install & run tests
npm install cypress
npx cypress run

# Tests require Docker containers running
docker-compose up -d
```

Tests validate: login flows, station creation, logbook operations, version checks.

### Common Tasks
- **Enable profiler**: Set `ENVIRONMENT = 'development'` in `index.php`
- **Routing**: CI3 maps `/controller/method` to `Controller::method` by default; `application/config/routes.php` is typically left at its default (`default_controller = 'dashboard'`). Only edit routes for custom remaps.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magicbug/Cloudlog](https://github.com/magicbug/Cloudlog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
