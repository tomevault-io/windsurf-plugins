---
trigger: always_on
description: This is a comprehensive GitHub repository that automatically generates a searchable Next.js website from a curated collection of AI benchmarks. The system parses a structured README.md file containing benchmark entries and creates a modern, responsive web interface for browsing and discovering AI benchmarks across Natural Language Processing, Computer Vision, and Multimodal domains.
---

# CLAUDE.md - Awesome AI Benchmarks Project

## Project Overview

This is a comprehensive GitHub repository that automatically generates a searchable Next.js website from a curated collection of AI benchmarks. The system parses a structured README.md file containing benchmark entries and creates a modern, responsive web interface for browsing and discovering AI benchmarks across Natural Language Processing, Computer Vision, and Multimodal domains.

## Architecture

### Repository Structure
```
awesome-ai-benchmarks/
├── README.md                    # Main benchmark data source
├── CONTRIBUTING.md              # Contribution guidelines
├── LICENSE                      # AGPL-3.0 license
├── .gitignore                   # Git ignore rules
├── .github/workflows/           # GitHub Actions
│   ├── build-and-deploy.yml     # Main deployment workflow
│   └── validate-readme.yml      # PR validation workflow
└── website/                     # Next.js application
    ├── package.json             # Dependencies and scripts
    ├── next.config.js           # Next.js configuration
    ├── tailwind.config.js       # Tailwind CSS configuration
    ├── tsconfig.json            # TypeScript configuration
    ├── postcss.config.js        # PostCSS configuration
    ├── src/
    │   ├── app/                 # Next.js 13+ app router
    │   │   ├── layout.tsx       # Root layout with SEO
    │   │   ├── page.tsx         # Homepage with search/filter
    │   │   └── globals.css      # Global styles
    │   ├── components/          # React components
    │   │   ├── BenchmarkCard.tsx    # Benchmark display card
    │   │   ├── SearchFilter.tsx     # Search and filtering UI
    │   │   ├── Header.tsx           # Site header with stats
    │   │   └── Footer.tsx           # Site footer
    │   ├── lib/                 # Utilities and types
    │   │   ├── benchmark-types.ts   # TypeScript interfaces
    │   │   ├── markdown-parser.ts   # README parsing logic
    │   │   └── utils.ts             # Helper functions
    ├── scripts/
    │   ├── markdown-parser.js      # Shared parsing logic module
    │   ├── parse-readme.js         # README to JSON parser
    │   └── validate-readme.js      # README format validator
    └── public/
        ├── data/                   # Benchmark data
        │   └── benchmarks.json     # Parsed benchmark data
        └── favicon.ico             # Site favicon
```

## Data Flow

### 1. Benchmark Entry Format
Benchmarks are defined in README.md using this exact structure:
```markdown
### [Category Name]

#### [Subcategory Name]
- **[Benchmark Name]** - [Brief description]
  - Paper: [URL to paper]
  - Code: [URL to code repository] (optional)
  - Website: [URL to official website] (optional)
  - Year: [Publication year]
  - Metrics: [Evaluation metrics] (optional)
  - Tags: [comma-separated tags] (optional)
```

### 2. Automated Processing Pipeline
1. **README Update** → GitHub Actions triggered
2. **Validation** → `validate-readme.js` checks format
3. **Parsing** → `parse-readme.js` extracts benchmark data
4. **Generation** → Creates `benchmarks.json` with structured data
5. **Build** → Next.js builds static site
6. **Deploy** → Vercel deploys updated website

### 3. Website Generation
- Static site generation for optimal performance
- Dynamic routing for individual benchmark pages
- Client-side search and filtering
- SEO optimization with structured data

## Key Components

### Frontend Components

#### BenchmarkCard.tsx
- Displays individual benchmark information
- Shows category, description, year, links
- Includes tags and metrics
- Responsive card layout

#### SearchFilter.tsx
- Real-time search functionality
- Category and subcategory filtering
- Advanced filters (year range, resource availability)
- Tag-based filtering
- Results count display

#### Header.tsx
- Project branding and statistics
- GitHub links and contribution CTAs
- Dynamic stats (total benchmarks, categories, etc.)
- Popular tags display

### Data Processing

#### markdown-parser.js
- Shared parsing logic module for README.md processing
- Contains `parseMarkdownToJSON()` function for converting markdown to structured JSON
- Includes `isValidUrl()` utility for URL validation
- Used by both parsing and validation scripts to ensure consistency
- Eliminates code duplication and provides single source of truth for parsing logic

#### parse-readme.js
- Node.js script that converts README.md to JSON using shared parsing logic
- Organizes data by categories and subcategories
- Generates structured JSON output for website consumption
- Usage: `npm run parse-data`

#### validate-readme.js
- Validates README.md format by parsing to JSON and validating structure
- Uses shared parsing logic from `markdown-parser.js`
- Checks required fields (name, description, paper or website, year)
- Validates URL formats and data integrity
- Reports errors and warnings with detailed feedback
- Usage: `npm run validate`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panilya/awesome-ai-benchmarks](https://github.com/panilya/awesome-ai-benchmarks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
