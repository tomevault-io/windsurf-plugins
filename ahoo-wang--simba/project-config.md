---
trigger: always_on
description: | Command | Description |
---

# AGENTS.md — Simba Wiki

## Build & Run Commands

| Command | Description |
|---------|-------------|
| `pnpm install` | Install dependencies |
| `pnpm run dev` | Start VitePress dev server |
| `pnpm run build` | Build static site |
| `pnpm run preview` | Preview built site |
| `pnpm run fix:mermaid` | Validate and fix Mermaid syntax |

## Project Structure

```
wiki/
├── .vitepress/
│   ├── config/          # VitePress configuration
│   │   ├── index.ts     # Main config (locales, search)
│   │   ├── en.ts        # English locale (nav, sidebar)
│   │   ├── zh.ts        # Chinese locale (nav, sidebar)
│   │   └── mermaid.ts   # Mermaid dark-mode theme
│   └── theme/
│       ├── index.ts     # Theme setup (mermaid-renderer)
│       └── custom.css   # Custom CSS (brand colors, mermaid)
├── scripts/
│   └── fix-mermaid.mjs  # Mermaid syntax validator/fixer
├── public/
│   └── logo.svg         # Site favicon/logo
├── index.md             # English home page
├── guide/               # Getting Started (EN)
├── architecture/        # Architecture (EN)
├── api/                 # API Reference (EN)
├── modules/             # Modules (EN)
├── testing/             # Testing (EN)
├── onboarding/          # Onboarding guides (EN)
├── zh/                  # Chinese translations
│   ├── index.md         # Chinese home page
│   ├── guide/           # Getting Started (ZH)
│   ├── architecture/    # Architecture (ZH)
│   ├── api/             # API Reference (ZH)
│   ├── modules/         # Modules (ZH)
│   ├── testing/         # Testing (ZH)
│   └── onboarding/      # Onboarding guides (ZH)
├── llms.txt             # LLM-friendly summary
└── llms-full.txt        # Full content for LLMs
```

## Content Conventions

- **Frontmatter**: Every `.md` file must have `title` and `description` in YAML frontmatter
- **Mermaid diagrams**: Use dark-mode colors (`#2d333b` fills, `#6d5dfc` borders, `#e6edf3` text)
- **Mermaid breaks**: Use `<br>` not `<br/>` (Vue compiler compatibility)
- **Sequence diagrams**: Always include `autonumber`
- **Citations**: Use `[file_path:line](https://github.com/Ahoo-Wang/Simba/blob/main/file_path#Lline)`
- **Bilingual**: English in root dirs, Chinese in `zh/` mirroring the same structure
- **Technical terms**: Keep class names, method names, config keys in English even in Chinese pages

## Documentation

- `llms.txt` — LLM-friendly project summary with wiki links
- `llms-full.txt` — Full page content for LLM context

## Boundaries

- ✅ Add new pages following the existing structure
- ✅ Run `pnpm run fix:mermaid` after editing Mermaid diagrams
- ✅ Run `pnpm run build` to verify before committing
- ⚠️ Ask before modifying `.vitepress/config/` files
- 🚫 Do not delete generated pages without discussion
- 🚫 Do not modify theme without testing

---
> Source: [Ahoo-Wang/Simba](https://github.com/Ahoo-Wang/Simba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
