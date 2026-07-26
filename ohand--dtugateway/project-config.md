---
trigger: always_on
description: When preparing to commit changes, automatically:
---

# dtuGateway Copilot Instructions

## Prepare for Commit Workflow

When preparing to commit changes, automatically:

1. Review uncommitted changes via `git status`
2. Update [CHANGELOG.md](../../CHANGELOG.md) with user-facing changes
3. Stage all modified files with `git add`
4. Generate a suggested commit message
5. Display staged changes summary

### Trigger Commands

- "prepare for commit"
- "prepare commit"
- "prep commit"
- "prepare for commit with changelog: [description]"

### Workflow Steps

1. **Get Current Changes**: Run `git status --short` to list all modified files
2. **Update CHANGELOG**: Add entry under `[Unreleased]` section with:
   - Category (Added/Changed/Fixed based on keywords in description)
   - User description or inferred summary
   - Timestamp
3. **Branch-Specific Actions**:
   - **If on `develop` branch**: Also update [.snapshot-summary.md](../../.snapshot-summary.md):
     - **Automatically analyze** code changes and create user-centric summaries (no developer prompts)
     - **Infer category** based on change type:
       - ✨ Features: New capabilities or functionality added
       - 🔧 Improvements: Enhancements to existing features
       - 🐛 Fixes: Bug fixes and corrections
     - **Generate entry** as one-liner (max ~80 chars), user-focused (NO technical jargon)
     - **Add to appropriate section** in .snapshot-summary.md
     - **Present for review**: Show updated summary to developer with message like: "I've added these changes to the snapshot summary — review and adjust if needed"
     - Developer accepts/modifies/removes entries before final commit
     - Stage updated .snapshot-summary.md with CHANGELOG
   - **If on `main` or other branches**: Skip snapshot summary
4. **Stage Files**: Run `git add .` to stage all changes including CHANGELOG and snapshot summary
5. **Generate Commit Message**: Create message using format:

   ```
   feat: [category] Short description of main change

   - Detailed bullet point 1
   - Detailed bullet point 2
   - Detailed bullet point 3

   Closes related issues if applicable
   ```

6. **Show Summary**: Display `git diff --cached --stat` to confirm staged changes

### Project Context

**Main Technologies**:

- Firmware: ESP32 (PlatformIO, C++ Arduino)
- Display: TFT_eSPI library, GC9A01 240×240 OLED LCD
- Web Dashboard: Alpine.js 3.x (PROGMEM served)
- Backend: ArduinoJson, MQTT via PubSubClient
- Build System: Python scripts for web header generation

**Key Directories**:

- `src/` — Firmware source (main .ino + implementation)
- `include/` — Headers + PROGMEM web assets
- `web_dev/src/` — Dashboard source (HTML, CSS, JS)
- `web_dev/build/` — Header generation scripts
- `doc/` — Documentation and screenshots

**Current Focus Areas**:

- Battery + Solar+Battery monitor modes
- TFT display rendering for remote monitoring
- Alpine.js UI for mode selection and control
- MQTT data integration

## Division of Work: Copilot vs GitHub Workflow

This snapshot release system uses **purposeful separation of responsibilities** to ensure both user-friendly release notes and reliable automation:

### Copilot's Job (BEFORE Commit on develop)

**Goal**: Create human-curated, user-centric summaries automatically

**When developer says "prepare for commit" on develop branch:**

1. Copilot reads actual code changes (git diff)
2. Copilot analyzes each change for user impact
3. Copilot **automatically generates** user-centric summary entries
4. Copilot adds to `.snapshot-summary.md`:
   - Categorizes as ✨ Features, 🔧 Improvements, or 🐛 Fixes
   - Writes one-liner (max 80 chars) in USER-FOCUSED language
   - NO technical jargon
5. Copilot presents summary to developer: _"I've added these changes to snapshot — review and adjust if needed"_
6. Developer accepts, modifies, or removes entries
7. Both CHANGELOG and `.snapshot-summary.md` staged together

**Example workflow:**

- Code change: Removed jQuery library from dashboard
- Copilot generates: `"Dashboard Faster & Offline — Removed jQuery CDN dependency"`
- Category: 🔧 Improvements
- Developer reviews and accepts ✓

**Result**: `.snapshot-summary.md` contains well-curated, user-friendly descriptions (developer-approved, not developer-created)

### Workflow's Job (DURING Push to GitHub)

**Goal**: Combine summaries with technical details; auto-detect for genericity

**When commit is pushed to develop branch (dev_build.yml triggers):**

1. **Read static summaries** from `.snapshot-summary.md` (already prepared by Copilot)
2. **Auto-detect latest main tag** (e.g., `v2.3.0018`) using: `git tag --list "v*" --sort=-version:refname --merged main`
3. **Get commit history** since that tag
4. **Combine both**:
   - User summaries (readable, curated by Copilot - PRIMARY)
   - Commit log (technical details, for developers - OPTIONAL, only if main tag found)
5. **Generate snapshot release notes** at GitHub

**Result**: Snapshot users see = User summaries (always) + Commit history (optional, if found)

**Important**: Release notes are ALWAYS user-centric. Commit history is omitted if main release tags can't be found (no error messages, no fallback technical content).

### Why This Distribution Works

| Responsibility                | Copilot                             | Workflow                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohAnd/dtuGateway](https://github.com/ohAnd/dtuGateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
