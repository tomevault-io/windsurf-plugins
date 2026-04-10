---
trigger: always_on
description: >-
---


# TYPO3 Content Blocks Development

> **Compatibility:** This skill targets **TYPO3 v14.x** with **Content Blocks 2.x**. Always match the [Packagist `friendsoftypo3/content-blocks`](https://packagist.org/packages/friendsoftypo3/content-blocks) constraint to your Core version.
> For **Content Blocks 1.x on TYPO3 v13**, upstream typically requires **TYPO3 ≥ 13.1** (not plain 13.0) — confirm on Packagist.
> Examples use TYPO3 v14 APIs and CB 2.x; adjust `composer.json` if upstream constraints differ.

> **TYPO3 API First:** Always use TYPO3's built-in APIs, core features, and established conventions before creating custom implementations. Do not reinvent what TYPO3 already provides. Always verify that the APIs and methods you use exist and are not deprecated in TYPO3 v14 by checking the official TYPO3 documentation.

## 1. The Single Source of Truth Principle

Content Blocks is the **modern approach** to creating custom content types in TYPO3. It eliminates redundancy by providing a **single YAML configuration** that generates:

- TCA (Table Configuration Array)
- Database schema (SQL)
- TypoScript rendering
- Backend forms and previews
- Labels and translations

### Why Content Blocks?

| Traditional Approach | Content Blocks Approach |
|---------------------|------------------------|
| Multiple TCA files | One `config.yaml` |
| Manual SQL definitions | Auto-generated schema |
| Separate TypoScript | Auto-registered rendering |
| Scattered translations | Single `labels.xlf` |
| Complex setup | Simple folder structure |

## 2. Installation

```bash
# Install via Composer (DDEV recommended)
ddev composer require friendsoftypo3/content-blocks

# After installation, clear caches
ddev typo3 cache:flush
```

### Security Configuration (Classic Mode)

For non-composer installations, deny web access to ContentBlocks folder:

```apache
# .htaccess addition
RewriteRule (?:typo3conf/ext|typo3/sysext|typo3/ext)/[^/]+/(?:Configuration|ContentBlocks|Resources/Private|Tests?|Documentation|docs?)/ - [F]
```

## 3. Content Types Overview

Content Blocks supports four content types:

| Type | Folder | Table | Use Case |
|------|--------|-------|----------|
| `ContentElements` | `ContentBlocks/ContentElements/` | `tt_content` | Frontend content (hero, accordion, CTA) |
| `RecordTypes` | `ContentBlocks/RecordTypes/` | Custom/existing | Structured records (news, products, team) |
| `PageTypes` | `ContentBlocks/PageTypes/` | `pages` | Custom page types (blog, landing page) |
| `FileTypes` | `ContentBlocks/FileTypes/` | `sys_file_reference` | Extended file references (photographer, copyright) |

## 4. Folder Structure

```
EXT:my_sitepackage/
└── ContentBlocks/
    ├── ContentElements/
    │   └── my-hero/
    │       ├── assets/
    │       │   └── icon.svg
    │       ├── language/
    │       │   └── labels.xlf
    │       ├── templates/
    │       │   ├── backend-preview.fluid.html
    │       │   ├── frontend.fluid.html
    │       │   └── partials/
    │       └── config.yaml
    ├── RecordTypes/
    │   └── my-record/
    │       ├── assets/
    │       │   └── icon.svg
    │       ├── language/
    │       │   └── labels.xlf
    │       └── config.yaml
    ├── PageTypes/
    │   └── blog-article/
    │       ├── assets/
    │       │   ├── icon.svg
    │       │   ├── icon-hide-in-menu.svg
    │       │   └── icon-root.svg
    │       ├── language/
    │       │   └── labels.xlf
    │       ├── templates/
    │       │   └── backend-preview.fluid.html
    │       └── config.yaml
    └── FileTypes/
        └── image-extended/
            ├── language/
            │   └── labels.xlf
            └── config.yaml
```

## 5. Creating Content Elements

### Kickstart Command (Recommended)

```bash
# Interactive mode
ddev typo3 make:content-block

# One-liner
ddev typo3 make:content-block \
  --content-type="content-element" \
  --vendor="myvendor" \
  --name="hero-banner" \
  --title="Hero Banner" \
  --extension="my_sitepackage"

# After creation, update database
ddev typo3 cache:flush -g system
ddev typo3 extension:setup --extension=my_sitepackage
```

### Predefined Basics (content elements)

List under `basics:` to pull in Core field groups: **`TYPO3/Header`**, **`TYPO3/Appearance`**, **`TYPO3/Links`**, **`TYPO3/Categories`**. See the [Content Blocks basics reference](https://docs.typo3.org/p/friendsoftypo3/content-blocks/main/en-us/API/Basics/Index.html).

### Minimal Content Element

```yaml
# EXT:my_sitepackage/ContentBlocks/ContentElements/hero-banner/config.yaml
name: myvendor/hero-banner
fields:
  - identifier: header
    useExistingField: true
  - identifier: bodytext
    useExistingField: true
```

### Full Content Element Example

```yaml
# EXT:my_sitepackage/ContentBlocks/ContentElements/hero-banner/config.yaml
name: myvendor/hero-banner
group: default
description: "A full-width hero banner with image and CTA"
prefixFields: true
prefixType: full
basics:
  - TYPO3/Appearance
  - TYPO3/Links
fields:
  - identifier: header
    useExistingField: true
  - identifier: subheadline
    type: Text
    label: Subheadline
  - identifier: hero_image
    type: File
    minitems: 1
    maxitems: 1
    allowed: common-image-types
  - identifier: cta_link
    type: Link
    label: Call to Action Link
  - identifier: cta_text
    type: Text
    label: Button Text
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dirnbauer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
