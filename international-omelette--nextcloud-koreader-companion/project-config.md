---
trigger: always_on
description: Prioritize substance, clarity, and depth. Challenge all my proposals, designs, and conclusions as hypotheses to be tested. Sharpen follow-up questions for precision, surfacing hidden assumptions, trade offs, and failure modes early. Default to terse, logically structured, information-dense responses unless detailed exploration is required. Skip unnecessary praise unless grounded in evidence. Explicitly acknowledge uncertainty when applicable. Always propose at least one alternative framing. Acce
---

# KOReader Companion - AI Agent Development Guide

## Your Mindset as a coding agent
Prioritize substance, clarity, and depth. Challenge all my proposals, designs, and conclusions as hypotheses to be tested. Sharpen follow-up questions for precision, surfacing hidden assumptions, trade offs, and failure modes early. Default to terse, logically structured, information-dense responses unless detailed exploration is required. Skip unnecessary praise unless grounded in evidence. Explicitly acknowledge uncertainty when applicable. Always propose at least one alternative framing. Accept critical debate as normal and preferred. Treat all factual claims as provisional unless cited or clearly justified. Cite when appropriate. Acknowledge when claims rely on inference or incomplete information. Favor accuracy over sounding certain. When citing, please tell me in-situ, including reference links. Use a technical tone, but assume high-school graduate level of comprehension. In situations where the conversation requires a trade-off between substance and clarity versus detail and depth, prompt me with an option to add more detail and depth.

## Overview

**What**: Nextcloud app providing OPDS 1.2 library + KOReader sync API
**Stack**: PHP 8.0+, Nextcloud 30-31, native NC APIs (IRootFolder, IConfig, IUserSession)
**Version**: 1.2.0 (Oct 2025)
**Repo**: https://github.com/international-omelette/nextcloud-koreader-companion

**Core capabilities**:
- OPDS catalog with HTTP Basic Auth (supports app passwords, LDAP, 2FA)
- KOReader sync (custom password, MD5 headers: `x-auth-user`, `x-auth-key`)
- Event-driven metadata extraction (EPUB/PDF)
- Web UI with infinite scroll, modal uploads
- Privacy-first: zero external dependencies

**Architecture shift (v1.0 → v1.2)**:
- Path-based → ID-based file operations
- Background indexing → Real-time event-driven processing
- Admin settings → User-level configuration

### System Design

```
Authentication:
  Web UI  → Nextcloud session
  OPDS    → HTTP Basic via IUserSession->logClientIn() (app passwords, LDAP, 2FA)
  KOReader→ MD5 header (`x-auth-user` + `x-auth-key`)

Data Flow:
  File Upload → FileCreationListener → BookService.extractMetadata()
             → Store in oc_koreader_metadata (key: file_id)
             → Generate hashes → Store in oc_koreader_hash_mapping
  OPDS Request → OpdsController → BookService.getBooks() → XML feed
  Sync Request → KoreaderController → Hash validation → oc_koreader_progress

Core Services:
  BookService            - Metadata extraction, pagination, real-time scanning
  PdfMetadataExtractor   - PDF-specific parsing (smalot/pdfparser)
  DocumentHashGenerator  - KOReader hash generation
  FilenameService        - Batch rename operations
```

**Directory map**:
```
lib/
├── Controller/
│   ├── PageController.php       # Web UI + settings API
│   ├── OpdsController.php       # OPDS 1.2 feeds (Basic Auth)
│   ├── KoreaderController.php   # Sync API (MD5 auth)
│   └── SettingsController.php   # User config (folder, auto-rename)
├── Service/
│   ├── BookService.php          # Core metadata + pagination
│   ├── PdfMetadataExtractor.php # Enhanced PDF support
│   ├── FilenameService.php      # Batch operations
│   └── DocumentHashGenerator.php# KOReader hashing
├── Listener/
│   ├── FileCreationListener.php # Real-time metadata extraction
│   └── FileDeleteListener.php   # Cleanup orphaned records
└── Migration/                   # Database schema (oc_koreader_*)

appinfo/routes.php  # API endpoint definitions
templates/page.php  # Main UI template
js/koreader.js      # CSP-compliant frontend (no inline handlers)
css/books.css       # Nextcloud-style responsive design
```

## Quick Reference

**Deploy & Test**:
```bash
./test_scripts/reset_and_deploy.sh       # Deploy to dev container
./test_scripts/test_opds.sh -v           # OPDS + Basic Auth validation
./test_scripts/test_koreader.sh -v       # KOReader sync compliance
```

**Key files**:
- `BookService.php:38` - Pagination logic
- `OpdsController.php` - OPDS feed generation
- `KoreaderController.php:updateProgress()` - Sync endpoint
- `routes.php` - All API paths
- `info.xml` - App metadata, NC version compat

**Database**:
- `oc_koreader_metadata` - Metadata cache (file_id primary key)
- `oc_koreader_hash_mapping` - Document hash mappings (auto-generated on upload)
- `oc_koreader_progress` - Reading positions
- `oc_preferences` - User settings (folder, password)

**Critical paths**:
- OPDS feed: `https://nc.example.com/apps/koreader_companion/opds`
- Sync API: `https://nc.example.com/apps/koreader_companion/sync/*`
- Signing certs: `~/.nextcloud/certificates/`

## Development Context

**Design philosophy**:
- **Cognitive load reduction** > clever abstractions
- **Self-documenting code** > comments
- **Linear logic** > complex conditionals

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [international-omelette/nextcloud-koreader-companion](https://github.com/international-omelette/nextcloud-koreader-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
