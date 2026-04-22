---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Environment

**Two-machine workflow:**
- **Vimes** (Linux) — development server where Claude Code runs. All code is written here in `/home/steveg/seonyx-holding/`.
- **Dibbler** (Windows) — test/run machine with Visual Studio 2022 Community and SSMS. Files are copied from Vimes to Dibbler for testing.

**Debug cycle:** Write code on Vimes → user copies files to Dibbler → user runs and pastes any errors back here. There is no remote execution from Vimes; all runtime errors come back as pasted text.

**Copy script convention:** Whenever a change touches more than one file, create or update `copy-updates.ps1` in the repo root so the user can run it on Dibbler to pull the files across. Use `copy-bookeditor.ps1` as the template — it has the right structure: source/destination variables, directory creation, separate new-files and modified-files arrays with coloured output, and a post-copy instructions block for any SQL scripts or manual steps needed.

```
src  \\192.168.69.75\seonyx-holding
dst  C:\Users\Steve\Dropbox\VITALIS\Seonyx\NEW_SITE_FEB26
```

## Build & Run

```bash
# Restore NuGet packages
nuget restore

# Build
msbuild Seonyx.csproj

# Database setup (run once, or to reset)
sqlcmd -S localhost -d Seonyx -i Database/schema.sql
```

Development server runs at `https://localhost:44327/` via IIS Express (port 62130 HTTP, 44327 HTTPS). Launch from Visual Studio with F5 on Dibbler.

## Configuration

Copy `Web.config.template` to `Web.config` and fill in:
- SQL Server connection string (`SeonyxContext` key)
- `AdminPasswordHash` — bcrypt hash of the admin password
- SMTP settings for contact form emails
- `BookEditorFilesPath` — defaults to `~/App_Data/BookEditorFiles/`

`Web.config` is gitignored; never commit it.

## Architecture

**Stack:** ASP.NET MVC 5 / .NET 4.6.2, Entity Framework 6, SQL Server, Razor views, vanilla JS. No npm, no webpack — bundling is ASP.NET's `Microsoft.AspNet.Web.Optimization`.

**Database approach:** Manual SQL migrations only. EF `SetInitializer` is disabled — no Code First migrations. Run scripts in `Database/` manually via `sqlcmd`. `SeonyxContext` in [Models/SeonyxContext.cs](Models/SeonyxContext.cs) is the EF DbContext.

**Authentication:** Forms Authentication (not ASP.NET Identity). Password hash stored in `Web.config` AppSettings. Login URL: `/admin/login`. Session timeout: 120 minutes.

**Route structure** (defined in [App_Start/RouteConfig.cs](App_Start/RouteConfig.cs)):
- `/` and public pages → `HomeController`, `PageController`, `DivisionController`, `LiteraryAgencyController`
- `/admin/...` → `AdminController` (pages, authors, books, content blocks, settings, submissions)
- `/admin/bookeditor/projects/...` → `BookProjectController`
- `/admin/bookeditor/files/...` → `FileUploadController`
- `/admin/bookeditor/editor/...` → `EditorController`
- `/admin/bookeditor/export/...` → `ExportController`
- `/{divisionSlug}/{pageSlug}` → `PageController` (catch-all for content pages)

## BookML XML Import — Active Major Change

The book editor is being transitioned from ad-hoc text file parsing to a formal XML interchange format. The full specification and XSD suite are in [Documentation/XML-transition-docs/](Documentation/XML-transition-docs/).

**Always read [BOOKML-SPEC.md](Documentation/XML-transition-docs/BOOKML-SPEC.md) before any importer work.** It explains the reasoning behind every decision.

### The Four Rules (non-negotiable)

1. **Validate before touching the database.** Run XSD validation against all three files (chapter, meta, notes). If any file fails, reject the entire import with a clear error. No partial imports.

2. **`pid` is sacred and immutable. `seq` is disposable.** The `pid` (e.g. `CH01-P0010`) is the permanent identity key assigned once at AI generation time and never changed. The `seq` attribute is just a display ordinal, freely rebalanced when gaps fall below 10. Never join on `seq`. Never use `seq` as a foreign key.

3. **Never overwrite existing draft records — append only.** The `paragraph_versions` table (and equivalent draft storage) is append-only. Importing a new draft inserts new rows at the new draft number; it never updates or deletes older draft rows.

4. **The three-file join is always on `pid`.** The chapter XML, meta XML, and notes XML for a chapter are always joined by `pid`. Never join on `seq`, position, line number, or any positional value.

### BookML File Structure

Each work contains:
- `book.xml` — manifest (title, authors, draft lineage, table of contents pointing to chapter/meta/notes files)
- `{chapter-id}/{chapter-id}-chapter.xml` — prose content; every paragraph has `pid` and `seq`
- `{chapter-id}/{chapter-id}-meta.xml` — sparse metadata (only paragraphs with noteworthy data); joined by `pid`
- `{chapter-id}/{chapter-id}-notes.xml` — annotations accumulate and are never deleted; resolved items flagged not removed; joined by `pid`

### PID Format

```
{CHAPTERID}-{TYPECODE}{SEQUENCE}
Examples: CH01-P0010  CH01-H0005  CH01-EP001  CH01-B0001
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Seonyx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
