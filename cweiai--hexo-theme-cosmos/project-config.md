---
trigger: always_on
description: Cosmos is an editorial Hexo theme for personal blogs, with a cover or article-list homepage, archives, taxonomy pages, and a configurable About page. The source repository is [cweiai/hexo-theme-cosmos](https://github.com/cweiai/hexo-theme-cosmos). It uses EJS templates, CommonJS helpers, plain CSS, and browser JavaScript. There is no frontend framework or asset compilation step.
---

# Agent Guide for Cosmos

## Overview

Cosmos is an editorial Hexo theme for personal blogs, with a cover or article-list homepage, archives, taxonomy pages, and a configurable About page. The source repository is [cweiai/hexo-theme-cosmos](https://github.com/cweiai/hexo-theme-cosmos). It uses EJS templates, CommonJS helpers, plain CSS, and browser JavaScript. There is no frontend framework or asset compilation step.

This guide applies throughout the repository. **MUST** means required, **SHOULD** means the default unless there is a concrete reason to depart, and **MAY** means optional. Follow explicit task instructions and applicable higher-priority instructions; surface a conflict instead of silently ignoring it.

## Read before working

Agents **MUST** read the root [README](README.md) and [contribution guide](CONTRIBUTING.md) before changing files. Read the relevant module guide and the sources listed below before editing that area. The contribution guide is the shared source for [code conventions](CONTRIBUTING.md#code-conventions), [commit conventions](CONTRIBUTING.md#commit-conventions), and [packaging](CONTRIBUTING.md#package-the-theme).

## Main Interfaces / Implementations

| Work area | Required reading and source of truth |
| --- | --- |
| Theme settings | [Configuration guide](docs/configuration.md), [_config.yml](_config.yml), [settings helpers](lib/README.md), and [reference generator](tools/reference.cjs) |
| Templates and page generation | [Layout guide](layout/README.md), [Hexo integration guide](scripts/README.md), and the relevant EJS templates/helpers |
| Articles, About, and custom pages | [Content guide](docs/content.md), [examples](examples/README.md), and [presets](presets/README.md) |
| Appearance and browser behavior | [Source guide](source/README.md), [style.css](source/css/style.css), and the affected templates/scripts |
| Translation and assets | [Language guide](languages/README.md), [icon guide](assets/social-icons/README.md), and [third-party notices](THIRD_PARTY_NOTICES.md) |
| Documentation | [Documentation guide](docs/README.md) and the corresponding English/Chinese document pair |
| Tests, installation, and distribution | [Test guide](test/README.md), [tool guide](tools/README.md), [automation guide](.github/AUTOMATION.md), [usage guide](docs/usage.md), and [package.json](package.json) |

## Art direction

Cosmos should feel like a warm printed journal: quiet, spacious, typographic, and focused on reading. Its character comes from asymmetric composition, large headings, thin rules, and generous whitespace. The About sidebar supports this editorial layout.

| Element | Default treatment |
| --- | --- |
| Paper and ink | Warm paper `#f4eedf`, dark brown ink `#342c24`, muted text `#6c6052` |
| Accents and rules | Terracotta `#a6402c`, honey `#eccc77`, fine rules `#d4c7b1` |
| Body and interface | Locally bundled Hanken Grotesk, with Chinese system-font fallbacks |
| Display accents | Locally bundled Petrona serif and italic, with Chinese serif fallbacks |
| Cover | Typography by default; optional static artwork supplied by the blog owner |

- Agents **MUST** preserve the default visual identity unless a task explicitly changes it. Extend the existing tokens and configuration before adding independent visual systems.
- Agents **SHOULD** favor readable text, deliberate spacing, and restrained transitions. Avoid unsolicited dashboard cards, glass effects, neon palettes, decorative animation, or remote font dependencies.
- Visual changes **MUST** retain responsive layouts, visible keyboard focus, semantic controls, and reduced-motion behavior. Primary navigation and article reading **MUST** remain usable without JavaScript; search and other enhancements may require it.
- New examples **MUST** use fictional or neutral content. Personal identities, live blog settings, and account details do not belong in theme defaults.

## Code and behavior contracts

- Agents **MUST** follow [.editorconfig](.editorconfig) and the contribution guide's code conventions. Keep changes focused and preserve neighboring style.
- Configuration **MUST** preserve the documented precedence: theme defaults, blog `_config.cosmos.yml`, then inline `theme_config`; About data and page overrides apply afterward. Objects merge recursively; arrays replace, including `[]`. Preserve meaningful `false` and empty-string overrides.
- New or changed settings **MUST** update `_config.yml`, both guides, and relevant examples/tests. Run `npm run docs:reference` to regenerate the marked default-table sections and [JSON schema](docs/theme.schema.json); do not hand-edit generated content.
- EJS **MUST** escape plain text and attributes with `<%=`. Use `<%-` only for intentionally rendered content, trusted HTML extension slots, or helpers that return safe markup. Reuse the existing URL, path, JSON, and CSS handling in [lib/settings.cjs](lib/settings.cjs) and [scripts/theme.js](scripts/theme.js).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cweiai/hexo-theme-cosmos](https://github.com/cweiai/hexo-theme-cosmos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
