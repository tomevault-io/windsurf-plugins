---
trigger: always_on
description: Self-hosted file sharing: Spring Boot 4.1 + Thymeleaf + SQLite (Flyway, Hibernate community dialect) + Tailwind. Java 21.
---

# QuickDrop — Agent Guide

Self-hosted file sharing: Spring Boot 4.1 + Thymeleaf + SQLite (Flyway, Hibernate community dialect) + Tailwind. Java 21.

## Code style
- **Comment only where the code is genuinely ambiguous** — a non-obvious constraint, a
  surprising "why", a trap someone would otherwise re-introduce. One or two terse lines.
- Don't restate what the code does, don't add rationale blocks to routine patterns, and
  don't explain what a previous version got wrong. That belongs in the commit message.
- Commit messages: subject line plus 2–3 short sentences.

## Build & run
- Build: `./mvnw clean package` (Windows: `mvnw.cmd clean package`). Output: `target/quickdrop.jar`.
- Run: `java -jar target/quickdrop.jar` → http://localhost:8080.
- Tailwind CSS: `npm run tw:build` rebuilds `src/main/resources/static/css/tailwind.css` from `tailwind-input.css`.
- Test suite: `./mvnw test` (JUnit/MockMvc, ~630 tests under `src/test/java/`) — runs on every CI build via the Jenkinsfile's `Build and Test` stage and gates Docker publish. Full testing guide (conventions, live e2e instances, browser testing, i18n): `docs/TESTING.md`. HTTP/security probes and browser exploratory testing are separate, on-demand passes (not part of the automatic CI gate) — see the same doc plus `scripts/seed_e2e.py` and the `quickdrop-e2e*` configs in `.claude/launch.json` for isolated live instances (ports 8081–8084). Write findings from such a pass to `docs/test-reports/` — that path stays gitignored because reports may describe unpatched vulnerabilities. Browser-only logic that MockMvc cannot reach has a second, much smaller suite: `npm test` (`node --test`, no framework dependency) over `src/test/js/`, currently covering the archive-layout rules in `upload/zip-builder.js`. `src/package.json` exists only to mark that tree as ES modules for Node; it sits above the Maven resource root and is not packaged.
- Logs at `log/quickdrop.log`, DB at `db/quickdrop.db`, uploads under `files/`, database backups under `db/backups/`, custom branding under `db/branding/` (all auto-created by [`QuickdropApplication`](src/main/java/org/rostislav/quickdrop/QuickdropApplication.java); see [`AppPaths`](src/main/java/org/rostislav/quickdrop/util/AppPaths.java)).
- Docker: `roastslav/quickdrop:latest`, exposes 8080; mount `/app/db`, `/app/files`, `/app/log` — backups and branding live under `/app/db`, so no separate volume is needed for either; omitting `/app/db` loses all three on container recreation.

## Architecture (follow the layers)
- Controllers (`controller/`): `FileViewController` (`/file/**` — upload form, list, detail, download, preview, history), `AdminViewController` (`/admin/**` — dashboard, file/paste management, settings, `/admin/links` merged short-links admin page), `FileRestController` (`/api/file/**` — chunked upload, upload status/abort, share APIs), `PasteViewController` (`/file/paste/**` — paste create/edit forms; paste viewing is still `FileViewController`), `ShareViewController` (`/share/{token}` — share landing page + key auth), `ShortLinkRestController` (`/api/link/**` — QR codes for any short link, `POST /api/link` creates a redirect link), `ShortLinkViewController` (`/link/new` — create page; `/s/{code}` and `/s/{code}/go` — resolver and interstitial-confirm for redirect links, upload-share links forward to `/share/{token}` unchanged), `StorageMigrationController` (`/admin/storage-migration**` — storage backend migration UI/API), `BackupController` (`/admin/backups**` — database backup schedule overview, on-demand backup, restore, delete, download), `PasswordViewController`, `IndexViewController`. Thymeleaf views in `src/main/resources/templates/`.
- Services (`service/`) own all business logic — there is no single do-everything service anymore. **Route file mutations through [`FileLifecycleService`](src/main/java/org/rostislav/quickdrop/service/FileLifecycleService.java)** (save/delete/extend/hide/share-token issuance) so cache eviction, history logging, and notifications fire via its `@CacheEvict`s and private `logHistory` helper. Its `generateShareToken`/`revokeShareToken` methods are thin wrappers that delegate the actual work to [`ShortLinkService`](src/main/java/org/rostislav/quickdrop/service/ShortLinkService.java) — kept in `FileLifecycleService` so no controller call site had to change when the short-link feature was carved out; add new upload-share-link logic to `ShortLinkService`, not here. Reads/authorization go through [`FileQueryService`](src/main/java/org/rostislav/quickdrop/service/FileQueryService.java) (`@Cacheable` list/detail queries). Streaming downloads and share-token redemption go through [`FileDownloadService`](src/main/java/org/rostislav/quickdrop/service/FileDownloadService.java), which does its own history logging/cache eviction for download events.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RoastSlav/quickdrop](https://github.com/RoastSlav/quickdrop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
