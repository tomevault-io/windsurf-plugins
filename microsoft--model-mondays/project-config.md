---
trigger: always_on
description: **Last Updated:** February 11, 2026
---

# Agent Guidelines for Model Mondays Repository

**Last Updated:** February 11, 2026  
**Repository:** microsoft/model-mondays  
**Branch:** 2026/s3-schedule-refresh

> **Custom Agent Available:** See [.github/copilot-agent.json](.github/copilot-agent.json) for the Model Mondays Maintainer agent configuration and [.github/skills/](.github/skills/) for reusable workflow skills.

> **Maintainer Guide:** See [.github/GUIDANCE.md](.github/GUIDANCE.md) for comprehensive maintainer guidance on using the agent, skills, and validation tools.

---

## 📋 Repository Overview

This repository hosts the **Model Mondays** livestream series and **Foundry Fridays** AMA sessions, providing educational content about AI models, Microsoft Foundry, and related technologies.

### Content Structure

```
model-mondays/
├── data/                           # JSON metadata files
│   ├── amas.json                   # AMA session metadata
│   ├── livestreams.json            # Episode metadata
│   ├── seasons.json                # Season information
│   ├── speakers.json               # Speaker profiles
│   ├── topics.json                 # Topic taxonomy
│   └── resources.json              # Learning resources
├── docs/
│   ├── model-mondays/              # Livestream episode posts (24 files)
│   ├── foundry-fridays/            # AMA session posts (34 files + README)
│   ├── customer-stories/           # Customer implementations
│   └── assets/                     # All images and banners
│       ├── model-mondays/          # Livestream banners (SX-EY.png)
│       ├── foundry-fridays/        # AMA banners (SX-EY-AMA.png)
│       ├── customer-stories/       # Customer story banners (SX-EY.png)
│       ├── people/                 # Speaker headshots
│       └── misc/                   # General assets
├── .2025/                          # Archived historical content
└── README.md                       # Main landing page
```

---

## 🎯 Key Conventions

### File Naming Patterns

1. **Episode Posts:** `yyyy-mm-dd-sXX-eYY.md`
   - Example: `2025-12-01-s03-e01.md` (Season 3, Episode 1, aired Dec 1, 2025)
   
2. **AMA Posts:** `yyyy-mm-dd-sXX-eYY.md`
   - Example: `2025-12-05-s03-e01.md` (Season 3, AMA 1, held Dec 5, 2025)
   
3. **Banners:**
   - Episode banners: `SX-EY.png` (e.g., `S3-E1.png`)
   - AMA banners: `SX-EY-AMA.png` (e.g., `S3-E1-AMA.png`)
   - Customer story banners: `SX-EY.png` (e.g., `S2-E6.png`)

### Content Standards

- **All image paths** use relative references: `../assets/[category]/filename.png`
- **Episode banners** → `docs/assets/model-mondays/`
- **AMA banners** → `docs/assets/foundry-fridays/`
- **Customer story banners** → `docs/assets/customer-stories/`
- **Banner images required** for all episode and AMA posts
- **Dates follow** ISO 8601 format: `YYYY-MM-DD`
- **Links are relative** within the repository (no absolute GitHub URLs for internal content)
- **Terminology:** Use "Microsoft Foundry" not "Azure AI Foundry" (URLs with `/azure/ai-foundry/` paths remain unchanged)

### Season Information

| Season | Episodes | AMAs | Timeline | Status |
|:---|:---:|:---:|:---|:---|
| Season 1 | 8 | 8 | Mar 2025 - May 2025 | Completed |
| Season 2 | 13 | 22 | Jun 2025 - Nov 2025 | Completed |
| Season 3 | 16 | 16 | Dec 2025 - Apr 2026 | Active |

---

## 🤖 Agent Responsibilities

### 1. Content Creation

When creating new episodes or AMAs:

1. **Check data files** in `data/` directory first for metadata
2. **Follow naming conventions** exactly (yyyy-mm-dd-sXX-eYY.md)
3. **Create/verify banner image** exists before linking
4. **Use relative paths** for all image references
5. **Update related files:**
   - Main `README.md` season tables
   - `docs/foundry-fridays/README.md` (for AMAs)
   - Relevant JSON metadata files in `data/`

### 2. Content Updates

When modifying existing content:

1. **Preserve file naming** - never change established filenames
2. **Update all cross-references** - episode posts link to AMAs and vice versa
3. **Verify image paths** remain valid after changes
4. **Check for broken links** in related content
5. **Update metadata files** in `data/` directory

### 3. Asset Management

When handling images and banners:

1. **Place in correct directory:**
   - Livestream banners → `docs/assets/model-mondays/`
   - AMA banners → `docs/assets/foundry-fridays/`
   - Customer story banners → `docs/assets/customer-stories/`
   - Speaker photos → `docs/assets/people/`
   - General assets → `docs/assets/misc/`

2. **Follow naming convention:**
   - Season/Episode format: `SX-EY.png` or `SX-EY-AMA.png`
   - Consistent capitalization (S and E uppercase)
   - Customer stories use same format as episodes: `SX-EY.png`

3. **Reference correctly:**
   - From episode posts: `../assets/model-mondays/S3-E1.png`
   - From AMA posts: `../assets/foundry-fridays/S3-E1-AMA.png`
   - From customer stories: `../assets/customer-stories/S2-E6.png`

### 4. Documentation Maintenance

**CRITICAL:** After making ANY changes to the repository:

1. **Update this AGENTS.md file** with:
   - New patterns or conventions introduced
   - Changes to directory structure
   - Updated statistics (episode counts, file counts, etc.)
   - New workflows or processes

2. **Run validation checks:**
   ```bash
   # Check for broken image links

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/model-mondays](https://github.com/microsoft/model-mondays) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
