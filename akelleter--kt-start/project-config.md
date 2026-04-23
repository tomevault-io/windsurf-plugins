---
trigger: always_on
description: Gestionnaire de favoris web auto-hébergé. Refonte complète de l'ancienne version (~10 ans, fichiers `.ini`) avec la même stack que KT-Drop.
---

# KT-Start — Contexte projet pour Claude Code

## Présentation
Gestionnaire de favoris web auto-hébergé. Refonte complète de l'ancienne version (~10 ans, fichiers `.ini`) avec la même stack que KT-Drop.

## Stack technique
- PHP 8.3+ (`declare(strict_types=1)` partout)
- SQLite 3 via PDO
- Bootstrap 5.3 + Bootstrap Icons 1.11
- `vlucas/phpdotenv` ^5.6
- SortableJS (CDN) pour le drag & drop
- Architecture MVC, front controller unique (`public/index.php`), router maison
- Aucun framework PHP externe

## Architecture
```
src/
├── Config/
│   ├── BadgeStyles.php     # 12 styles de badge avec gradient() — deepBlue, turquoise, etc.
│   └── Config.php          # Accès à $_ENV
├── Controller/
│   ├── AdminController.php # index, usersPage, listsPage, foldersPage, settingsPage, backupPage, maintenancePage, tagsPage, statsPage, tagRename, tagDelete, userStore/Update/Delete, listStore/Rename/Delete/SetDefault, adminFolderStore/Rename/Delete/Reorder, settingUpdate, runMigration, exportBookmarks, exportFull, importBookmarks
│   ├── AuthController.php  # login, logout → redirige vers ?action=home
│   └── BookmarkController.php  # home(), index(), store(), update(), delete(), reorder(), fetchMeta(), checkDuplicate(), linksReport(), checkSingleLink(), deleteDeadLinks(), resetLinkStatus(), followRedirect(), folderStore(), folderRename(), folderDelete(), explorerReorder(), bookmarklet(), bookmarkletStore()
├── Core/
│   ├── Auth.php            # Session : ktstart_session (≠ ktdrop_session), isAdmin()
│   ├── Csrf.php
│   ├── Database.php        # Singleton PDO SQLite
│   ├── Flash.php
│   ├── Response.php
│   ├── Router.php          # Routes par ?action=xxx, non-auth → ?action=home
│   └── View.php            # render(), renderRaw() (sans layout — popup bookmarklet), e(), asset() → 'public/assets/...'
├── Repository/
│   ├── BookmarkRepository.php  # findPublic(), findPublicByList(), findFiltered(), findByUserAndListWithFolder(), CRUD, reorder(), setFolderAndPosition(), getAllTags(), getAllTagsAdmin(), renameTag(), deleteTag(), deleteTagsUsedOnce(), findByUrl(), findAllByUser(), updateCheckStatus(), resetCheckStatus(), updateUrl(), deleteMultiple(), countDeadLinksAll()
│   ├── FolderRepository.php    # findAllByUser(), findAllByUserInList(), findAllByListId(), findAllByListIdWithUser(), findById(), existsForUserInList(), create(), rename(), renameAdmin(), setParentAndPosition(), setParentAndPositionAdmin(), wouldCreateCycle(), wouldCreateCycleAdmin(), deleteAndLiftChildren(), deleteAndLiftChildrenAdmin(), countAll(), groupByParent()
│   ├── ListRepository.php      # findAll(), findByName(), create(), rename(), findAllWithCount(), findDefault(), setDefault(), clearDefault()
│   ├── SettingsRepository.php  # get(), set(), all() — table settings clé/valeur
│   ├── StatsRepository.php     # overview(), perUser(), perList(), perLinkStatus(), topTags(), perMonth(), perBadgeStyle()
│   └── UserRepository.php      # CRUD complet, emailExists()
└── Service/
    ├── ImportExportService.php # export() v1 favoris, exportFull() v2 backup, import() détection auto v1/v2
    ├── MigrationService.php    # Migrations idempotentes : PRAGMA + ALTER TABLE ADD COLUMN
    ├── UrlCheckService.php     # check() HEAD/GET cURL, statuts ok/redirect/error/timeout, getFinalUrl() suit les redirections, proxy DB→.env
    └── UrlMetaService.php      # curl + DOMDocument → title/host/description
```

## Schéma SQLite
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL DEFAULT 'admin',
    created_at TEXT NOT NULL
);

CREATE TABLE lists (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    is_default INTEGER NOT NULL DEFAULT 0,  -- 1 seule liste par défaut max
    created_at TEXT NOT NULL
);

CREATE TABLE bookmarks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    url TEXT NOT NULL,
    host TEXT,
    title TEXT,
    description TEXT,
    badge_style TEXT NOT NULL DEFAULT 'deepBlue',
    badge_text TEXT NOT NULL DEFAULT '',
    tags TEXT,           -- virgule-séparé ex: "php,dev"
    visibility TEXT NOT NULL DEFAULT 'private',  -- 'public' | 'private'
    list_id INTEGER REFERENCES lists(id),
    folder_id INTEGER REFERENCES folders(id),  -- dossier optionnel dans la liste
    user_id INTEGER NOT NULL REFERENCES users(id),
    position INTEGER DEFAULT 0,
    created_at TEXT NOT NULL,
    last_check_status TEXT,   -- 'ok' | 'redirect' | 'error' | 'timeout' | NULL (non vérifié)
    last_check_at TEXT        -- datetime de la dernière vérification
);

CREATE TABLE folders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    user_id INTEGER NOT NULL REFERENCES users(id),
    list_id INTEGER NOT NULL REFERENCES lists(id),
    parent_id INTEGER REFERENCES folders(id),  -- hiérarchie parent/enfant
    position INTEGER NOT NULL DEFAULT 0,
    created_at TEXT NOT NULL
);

CREATE TABLE settings (
    key   TEXT PRIMARY KEY,
    value TEXT NOT NULL
    -- ex: bookmarks_per_page = '24'
);
```

## Routing
Toutes les routes passent par `?action=xxx` :
- `home` (public) → favoris publics, non connecté
- `login` / `login_submit` (public)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aKelleter) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
