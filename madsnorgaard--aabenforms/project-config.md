---
trigger: always_on
description: **ÅbenForms** is a headless workflow automation platform for Danish municipalities, built on:
---

# ÅbenForms Backend - Drupal 11

## Project Overview

**ÅbenForms** is a headless workflow automation platform for Danish municipalities, built on:
- **Backend**: Drupal 11.3.10 (this repository)
- **Frontend**: Nuxt 3 (separate repository)
- **Deployment**: contabo-infrastructure orchestration (separate repository, deploys to VPS2)

This backend provides:
- JSON:API endpoints for headless content delivery
- ECA (Event-Condition-Action) workflow engine (modern replacement for Maestro)
- Multi-tenancy via Domain module
- Danish government service integrations (MitID, Serviceplatformen, Digital Post)
- GDPR-compliant data handling with field-level encryption
- Modular SF1601 Digital Post (`aabenforms_digital_post` + ECA bridge submodule, plug-and-play on bare Drupal 11)
- Shared admin design tokens via `aabenforms_core/admin` library (CSS custom properties)

## Active workstream

Modular Digital Post + NemLogin rewrite. Goal: each Danish-government integration installs cleanly on any modern Drupal 11 with at most one mainstream contrib (`drupal:key`), with no dependency on proprietary alternative stacks.

- **Session 1 (shipped)**: `aabenforms_digital_post` core (DTOs, sender service, fake_db / wiremock transports, Drush, settings form, log table). Runs on the demo in `fake_db` mode. No live MeMo/SOAP and no idempotency yet.
- **Session 2A (shipped)**: `aabenforms_digital_post_eca` submodule (plugin id `aabenforms_digital_post_send`). `citizen_service_application.bpmn` Approved + Rejected branches both wired.
- **Session 2B (planned, issue #77)**: real MeMo XML + SF1601 SOAP via `itk-dev/serviceplatformen`; `live_test` against Serviceplatformen exttest endpoint.
- **Session 2C (planned, issue #79)**: `aabenforms_nemlogin` OIDC core + Keycloak preset + shim `aabenforms_mitid` over it. These modules do not exist yet.
- **Session 3 (planned)**: webform / beskedfordeler (#78) / key bridges, advanced queue, examples, remove `aabenforms_mitid`, bare-D11 GitHub Actions verification.

Recent platform-wide fixes:
- `aabenforms_log` shim replaces removed upstream `eca_base_log`/`eca_base_mail`. `aabenforms_workflows_update_11001` migrates saved configs.
- `hook_storage_transform_import` in `aabenforms_workflows.module` preserves wizard-created configs across `drush cim` (without it, every deploy nuked admin-created workflows).
- Wizard step indicator modernized (numbered circles + track line) using `--af-*` tokens from `aabenforms_core/admin`.
- TemplateBrowserController preview thumbnails fixed (controller now emits `data-xml` on canvas) + Active Workflows section moved on top when non-empty.
- TemplateSelectForm: `disableCache()` to dodge File-object serialization on rebuild + path-traversal hardening on import.

## Technology Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| Drupal Core | 11.3.10 | CMS foundation |
| PHP | 8.4 | Runtime |
| MariaDB | (DDEV default) | Database (consistent local to production) |
| DDEV | Latest | Local development |
| Composer | 2.x | Dependency management |
| Drush | 13.7 | CLI tool |

## Essential Commands

### DDEV Operations
```bash
# Start environment
cd /home/mno/ddev-projects/aabenforms/backend
ddev start

# Stop environment
ddev stop

# SSH into container
ddev ssh

# Database operations
ddev drush sql:dump > backup.sql
ddev import-db --file=backup.sql
```

### Drush Commands
```bash
# Clear cache
ddev drush cr

# Export/Import configuration
ddev drush config:export -y
ddev drush config:import -y

# Check database updates
ddev drush updatedb -y

# User operations
ddev drush user:login    # Generate one-time login link

# Module operations
ddev drush pm:enable aabenforms_tenant
ddev drush pm:uninstall <module_name>
```

### Composer Operations
```bash
# Add module
ddev composer require drupal/<module_name>

# Remove module
ddev composer remove drupal/<module_name>

# Update all dependencies
ddev composer update

# Security updates
ddev composer update drupal/core --with-dependencies
```

## Module Architecture

### Custom Modules (Package: ÅbenForms)

These are the modules that actually exist today under `web/modules/custom/`:

```
web/modules/custom/
├── aabenforms_core/              # Foundation: shared services, Serviceplatformen
│   │                            #   client, field-level CPR encryption (AES-256),
│   │                            #   GDPR audit logging, admin design tokens
│   ├── aabenforms_core.info.yml
│   ├── aabenforms_core.module
│   └── src/
│
├── aabenforms_tenant/            # Multi-tenancy (depends on: domain)
│   ├── aabenforms_tenant.info.yml
│   └── src/
│
├── aabenforms_workflows/         # ECA integration + Workflow Modeler editor
│   │                            #   13 workflow templates (.bpmn source files),
│   │                            #   21 ECA action plugins (incl. CPR/CVR lookup)
│   ├── aabenforms_workflows.info.yml
│   ├── aabenforms_workflows.module
│   └── workflows/                # .bpmn template source files
│
├── aabenforms_webform/           # Custom webform elements with server-side
│   └──                          #   validation: CPR (modulus-11), CVR, Adressevælger address
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madsnorgaard/AabenForms](https://github.com/madsnorgaard/AabenForms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
