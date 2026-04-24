---
trigger: always_on
description: ⚠️ IMPORTANT: This file is partially managed by Monoco.
---

<!--
⚠️ IMPORTANT: This file is partially managed by Monoco.
- Content between MONOCO_GENERATED_START and MONOCO_GENERATED_END is auto-generated.
- Use `monoco sync` to refresh this content.
- Do NOT manually edit the managed block.
- Do NOT add content after MONOCO_GENERATED_END (use separate files instead).
-->

<!-- MONOCO_GENERATED_START -->
## Monoco

> **Auto-Generated**: This section is managed by Monoco. Do not edit manually.

### Doc-Extractor

#### Doc-Extractor: Document Normalization and Rendering

Document extraction and rendering system for converting various document formats into standardized WebP page sequences suitable for VLM (Vision Language Model) consumption.

##### Overview

Doc-Extractor provides a content-addressed storage system for documents with automatic format normalization:

- **Input**: PDF, DOCX, PPTX, XLSX, Images (PNG, JPG), Archives (ZIP, TAR, RAR, 7Z)
- **Output**: WebP page sequences with configurable DPI and quality
- **Storage**: SHA256-based content addressing in `~/.monoco/blobs/`

##### Commands

###### Extract Document
```bash
monoco doc-extractor extract <file> [options]
```

Options:
- `--dpi, -d`: DPI for rendering (72-300, default: 150)
- `--quality, -q`: WebP quality (1-100, default: 85)
- `--pages, -p`: Specific pages to render (e.g., "1-5,10,15-20")

###### List Extracted Documents
```bash
monoco doc-extractor list [--category <cat>] [--limit <n>]
```

###### Search Documents
```bash
monoco doc-extractor search <query>
```

###### Show Document Details
```bash
monoco doc-extractor show <hash_prefix>
monoco doc-extractor cat <hash_prefix>    # Show metadata JSON
monoco doc-extractor source <hash_prefix> # Show source/archive info
```

###### Index Management
```bash
monoco doc-extractor index rebuild   # Rebuild index from blobs
monoco doc-extractor index stats     # Show index statistics
monoco doc-extractor index clear     # Clear index (keeps blobs)
monoco doc-extractor index path      # Show index file path
```

###### Cleanup
```bash
monoco doc-extractor clean [--older-than <days>] [--dry-run]
monoco doc-extractor delete <hash_prefix> [--force]
```

##### Storage Structure

```text
~/.monoco/blobs/
├── index.yaml              # Global metadata index
└── {sha256_hash}/          # Content-addressed directory
    ├── meta.json           # Document metadata
    ├── source.{ext}        # Original file (preserved extension)
    ├── source.pdf          # Normalized PDF format
    └── pages/
        ├── 0.webp          # Page 0 rendering
        ├── 1.webp          # Page 1 rendering
        └── ...
```

##### Python API

```python
from monoco.features.doc_extractor import DocExtractor, ExtractConfig

extractor = DocExtractor()
config = ExtractConfig(dpi=150, quality=85)
result = await extractor.extract("/path/to/document.pdf", config)

print(f"Hash: {result.blob.hash}")
print(f"Pages: {result.page_count}")
print(f"Cached: {result.is_cached}")
```

##### Key Principles

1. **Content-Addressed**: Files are stored by SHA256 hash - duplicates are automatically deduplicated
2. **Format Normalization**: All documents are converted to PDF then rendered to WebP
3. **Archive Support**: ZIP and other archives are automatically extracted, with inner document tracked
4. **Cache-Aware**: Extraction results are cached; re-extracting returns cached results instantly

### Agent

##### Monoco Core

Core commands for project management. Follows the **Trunk Based Development (TBD)** pattern.

- **Init**: `monoco init` (Initialize a new Monoco project)
- **Config**: `monoco config get|set <key> [value]` (Manage configuration)
- **Sync**: `monoco sync` (Sync with agent environment)
- **Uninstall**: `monoco uninstall` (Clean up agent integration)

---

#### ⚠️ Agent Must Read: Git Workflow Protocol (Trunk-Branch)

Before modifying any code, **must** follow these steps:

##### Standard Process

1. **Create Issue**: `monoco issue create feature -t "Feature Title"`
2. **🔒 Start Branch**: `monoco issue start FEAT-XXX --branch`
   - ⚠️ **Isolation Required**: Use `--branch` or `--worktree` parameter
   - ❌ **Trunk Operation Prohibited**: Forbidden from directly modifying code on Trunk (`main`/`master`)
3. **Implement**: Normal coding and testing
4. **Sync Files**: `monoco issue sync-files` (must run before submitting)
5. **Submit for Review**: `monoco issue submit FEAT-XXX`
6. **Merge to Trunk**: `monoco issue close FEAT-XXX --solution implemented`

##### Quality Gates

- Git Hooks automatically run `monoco issue lint` and tests
- Do not use `git commit --no-verify` to bypass checks
- Linter prevents direct modifications on protected Trunk branches

> 📖 See `monoco-issue` skill for complete workflow documentation.

### Issue Management

#### Issue Management & Trunk Based Development

Monoco follows the **Trunk Based Development (TBD)** pattern. All development occurs in short-lived branches (Branch) and is eventually merged back into the main line (Trunk).

System for managing task lifecycles using `monoco issue`.

- **Create**: `monoco issue create <type> -t "Title"`
- **Status**: `monoco issue open|close|backlog <id>`
- **Check**: `monoco issue lint`
- **Lifecycle**: `monoco issue start|submit|delete <id>`
- **Sync Context**: `monoco issue sync-files [id]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IndenScale/Typedown](https://github.com/IndenScale/Typedown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
