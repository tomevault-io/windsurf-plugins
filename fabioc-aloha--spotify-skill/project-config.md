---
trigger: always_on
description: This project is an **example of Claude Desktop Skills** - specifically demonstrating how to create production-ready skills that extend Claude's capabilities. The Spotify API Skill serves as a comprehensive reference implementation for building API integration skills.
---

# Copilot Instructions - Spotify Skill Project

## Project Overview

This project is an **example of Claude Desktop Skills** - specifically demonstrating how to create production-ready skills that extend Claude's capabilities. The Spotify API Skill serves as a comprehensive reference implementation for building API integration skills.

### What This Project Is

- **Primary Purpose**: Educational example and template for creating Claude Desktop Skills
- **Secondary Purpose**: Production-ready Spotify API integration skill
- **Audience**: Developers learning to create custom Claude skills
- **Status**: Complete, production-ready, fully documented

### Project Type

This is a **Complete Skill Development Toolkit** that includes:

1. A complete working skill (Spotify API integration)
2. **Automated development tools** (init, validate, package) - NEW!
3. **Official specifications** (Agent Skills Spec v1.0) - NEW!
4. Comprehensive guides for creating any type of skill
5. Interactive workbooks and templates
6. Advanced patterns and examples
7. Best practices and troubleshooting

---

## What Are Claude Desktop Skills?

### Definition

**Skills** are modular, self-contained packages that extend Claude's capabilities by providing:

- Specialized workflows for specific domains
- Tool integrations (APIs, file formats, services)
- Domain expertise and procedural knowledge
- Bundled resources (scripts, references, assets)

**Official Documentation**: For up-to-date information on creating custom skills, see:

- [Official Claude Skills Documentation](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills)
- [Official Skills Examples Repository](https://github.com/anthropics/skills)

### Skill Architecture

```
skill-name/
├── SKILL.md                    # Required: Main documentation with YAML frontmatter
├── scripts/                    # Optional: Executable Python/JS code
│   ├── main_script.py
│   └── helper_script.py
├── references/                 # Optional: Reference documentation
│   ├── api_reference.md
│   └── authentication_guide.md
└── assets/                     # Optional: Templates, images, etc.
    └── template.docx
```

### Core Principles of Skill Creation

1. **Concise is Key** - Context window is shared; only include what Claude doesn't know
2. **Set Appropriate Degrees of Freedom** - Match specificity to task complexity
3. **Progressive Disclosure** - Load content as needed (metadata → SKILL.md → resources)
4. **Single Source of Truth** - No duplicated information across files

---

## Project Structure

### Main Components

```
Spotify-Skill/
├── README.md                   # Main repository documentation
├── SPOTIFY_SKILL_README.md     # Complete project overview
├── USER_GUIDE.md               # Complete usage guide
├── QUICK_START.md              # 5-minute setup guide
├── agent_skills_spec.md        # Official Agent Skills Spec v1.0
├── spotify-api.skill           # Packaged skill (git ignored)
│
├── .github/                    # Repository assets
│   ├── copilot-instructions.md # AI assistant instructions
│   ├── banner-dark.svg         # Dark theme banner
│   └── banner-light.svg        # Light theme banner
│
├── tools/                      # Skill development utilities
│   ├── init_skill.py           # Create new skills from template
│   ├── validate_skill.py       # Validate skill structure
│   ├── package_skill.py        # Package skills for distribution
│   └── README.md               # Tool documentation
│
├── examples/                   # Curated skill examples
│   ├── README.md               # Pattern comparison and selection guide
│   └── EXAMPLES_REFERENCE.md   # Detailed analysis of 6 example patterns
│
├── spotify-api/                # Skill source files
│   ├── SKILL.md                # Main skill documentation (REQUIRED)
│   ├── .env                    # Spotify credentials (git ignored)
│   ├── scripts/
│   │   ├── spotify_client.py  # Core API wrapper (40+ methods)
│   │   └── playlist_creator.py # High-level playlist utilities
│   └── references/
│       ├── api_reference.md    # Spotify API reference
│       └── authentication_guide.md # OAuth 2.0 setup
│
└── Guide/                      # Skill creation education
    ├── 00_START_HERE.md        # Entry point for learning
    ├── INDEX.md                # Navigation guide
    ├── SKILL_CREATION_GUIDE.md # Complete skill creation guide
    ├── SKILL_CREATION_WORKBOOK.md # Interactive planning templates
    ├── ADVANCED_SKILL_EXAMPLES.md # Patterns and examples
    └── SPOTIFY_CREDENTIALS_SETUP.md # Credential setup guide
```

---

## Guide Folder Knowledge

The `Guide/` folder contains the **complete educational curriculum** for skill creation:

### 1. Navigation Documents

**`00_START_HERE.md`** - Primary entry point

- Explains what's in the package
- Provides learning paths based on goals
- Recommends reading order
- Estimated time commitments

**`INDEX.md`** - Comprehensive navigation

- Quick reference table for all documents
- Goal-based navigation ("I want to...")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabioc-aloha/spotify-skill](https://github.com/fabioc-aloha/spotify-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
