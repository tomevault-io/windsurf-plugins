---
trigger: always_on
description: This file is the entry point when working on the **YForm addon itself** (not when consuming it from a project). It collects what's not obvious from grepping: the architectural model, the lifecycle, the cache contracts, the do-not-touch zones.
---

# CLAUDE.md — YForm Addon

This file is the entry point when working on the **YForm addon itself** (not when consuming it from a project). It collects what's not obvious from grepping: the architectural model, the lifecycle, the cache contracts, the do-not-touch zones.

## Was ist YForm?

YForm ist ein REDAXO-AddOn mit zwei Aufgaben:

1. **Frontend-Formularbau** — Pipe-Syntax oder PHP-API, validierung, Aktionen (DB-Insert, Mailversand, Redirect).
2. **Tabellenverwaltung im Backend** — visuell zusammengeklickte Tabellen + Backend-UI + ORM (`rex_yform_manager_dataset` / YOrm).

Beides nutzt denselben Form-Pipeline-Runtime (`rex_yform`). Der Backend-Datasatz-Save geht durch den gleichen Field/Validate/Action-Loop wie ein Frontend-Submit, nur „headless".

## Vorhandene Skills nutzen

Es gibt bereits gut gepflegte Skills für die Anwender-Sicht. Bei Aufgaben, die nur das „Konsumieren" von YForm betreffen (Form bauen, Dataset abfragen, REST-Endpoint registrieren), **erst dort nachschauen**:

| Skill (im `redaxo-yform`-Plugin) | Wofür |
|---|---|
| `redaxo-yform` (Top-level Skill) | Schnellreferenz quer durch alle Themen |
| `yform-tables` | Tabellen in `install.php`, Schema-Migrationen, Tablesets |
| `yform-fields` | Alle Value/Validate/Action-Typen + Pipe-Syntax |
| `yform-frontend` | Public Forms, CSRF, Uploads, Spam-Schutz, objparams |
| `yform-datasets` | YOrm-Queries, Joins, Pagination, Relations |
| `yform-email-templates` | Email-Templates, Platzhalter, programmatischer Versand |
| `yform-rest-api` | REST-Routen, Token-Auth, CORS, Field-Whitelists |

Bei Arbeit **am AddOn selbst** (interne Klassen ändern, Field-Typen hinzufügen, EPs erweitern, Schema migrieren) reichen die Skills nicht. Dafür gibt es die `references/` unten.

## Architektur in 60 Sekunden

```
HTTP-Request (Frontend/Backend/REST)
    │
    ▼
boot.php
    ├── lädt Template-Pfade
    ├── injiziert Backend-Assets (CSS, JS, Inputmask, Daterangepicker)
    ├── kompiliert SCSS bei `compile: 1`
    ├── PACKAGES_INCLUDED → rex_yform_rest::handleRoutes() (nur Frontend)
    ├── registriert complex_perms `yform_manager_table_{view,edit}`
    ├── baut Backend-Pages für jede aktive Tabelle
    ├── verdrahtet MEDIA_IS_IN_USE, YFORM_SAVED (History), Cron
    │
    ▼
Pages (pages/manager.*.php) ──► rex_yform_manager ──► rex_yform_manager_dataset
                                                          │
                                                          ▼
                                                    interner rex_yform (headless)
                                                          │
                                                          ▼
                                                    executeFields → executeActions
                                                          │
                                                          ▼
                                                    action|db (INSERT/UPDATE)
                                                          │
                                                          ▼
                                                    YFORM_SAVED EP
                                                          │
                                                          ▼
                                                    History-Snapshot (wenn aktiv)
```

System-Tabellen (alle in `install.php` per `rex_sql_table::ensure()`):

- `rex_yform_table` — Tabellen-Metadaten
- `rex_yform_field` — Field/Validate/Action-Zeilen
- `rex_yform_history` + `rex_yform_history_field` — Snapshots
- `rex_yform_email_template` — Email-Templates
- `rex_yform_rest_token` + `rex_yform_rest_token_access` — REST-Auth

Alle gemanagten User-Tabellen werden separat (per `setTable()` + manuelles `rex_sql_table::ensure()`) angelegt. YForm besitzt sie nicht — es kennt nur ihre Konfiguration.

## Lifecycle eines Formulars

`rex_yform::getForm()` ruft `executeFields()` → `executeActions()`:

1. **`initializeFields()`** — CSRF-Feld prepend, dann jede `form_elements`-Zeile als `rex_yform_value_*` / `rex_yform_validate_*` / `rex_yform_action_*` instanziieren. `loadParams()` bindet die `objparams` per Referenz, d.h. jedes Feld kann den shared State mutieren.
2. **Value-Population**: REQUEST → sql_object (bei `getdata=true`) → objparams.data → fixdata. Spätere Quellen überschreiben frühere.
3. **`preValidateAction` → Validate-Phase (nur bei `send=1`) → `postValidateAction`**.
4. **Value-`enterObject`** — rendert HTML in `form_output[$id]`, schreibt in `value_pool.email` und `value_pool.sql` (wenn `saveInDb()`).
5. **`postValueAction` (nur send=1) → `postFormAction`**.
6. **Action-Phase (nur send=1 und keine warnings)**: `preAction` → `executeAction` → `postAction`. `action|db` zieht aus `value_pool.sql`, `action|tpl2email` aus `value_pool.email`.

Detaillierter Ablauf siehe **`.claude/references/02-form-pipeline.md`**.

## Was im Repo **nicht** geändert werden darf

- **`ytemplates/bootstrap/*.tpl.php`** — wird bei jedem Update überschrieben. Override stattdessen in Projekt-AddOns via `rex_yform::addTemplatePath()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yakamara/yform](https://github.com/yakamara/yform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
