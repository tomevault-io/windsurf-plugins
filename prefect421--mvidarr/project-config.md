---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Memories

- Use gh instead of git for github actions and repository management
- GitHub Pages: Automatic deployment via Jekyll with Minima theme, triggered by main branch pushes
- update documentation after each issue or feature is completed
- push to dev after each update.
- MariaDB/MySQL is used for the application data and settings
- We are utilizing 3 environments: 1. Dev is running on local server port 5000. 2. Docker is running on local machine docker install on port 5001. 3. Prod is running on 192.168.1.132:5050 in a docker install.
- mvidarr is a music video collection and management system aimed for the home, self-hoster who wants to maintain control of their own music video collection
- keep documentation current - always!
- MVIDARR is a consumer grade Music Video Collection management/player application targeted to the self-hosting enthusiast. It should be able to be ran as a service or a docker.
- ask before planning anything enterpise grade, it may not be needed.

## Critical Thinking and Feedback

### IMPORTANT: Always critically evaluate and challenge user suggestions, even when they seem reasonable.

- ** USE BRUTAL HONESTY: Don't try to be polite or agreeable. Be direct, challenge assumptions, and point out flaws immediately.
- ** Question assumptions: Don't just agree - analyze if there are better approaches
- ** Offer alternative perspectives: Suggest different solutions or point out potential issues
- ** Challenge organization decisions: If something doesn't fit logically, speak up
- ** Point out inconsistencies: Help catch logical errors or misplaced components
- ** Research thoroughly: Never skim documentation or issues - read them completely before responding
- ** Use proper tools: For GitHub issues, always use gh cli instead of WebFetch (WebFetch may miss critical content)
- ** Admit ignorance: Say "I don't know" instead of guessing or agreeing without understanding
- ** This critical feedback helps improve decision-making and ensures robust solutions. Being agreeable is less valuable than being thoughtful and analytical.
- ** you are an expert website developer, act like it.

### Example Behaviors

-    ✅ "I disagree - that component belongs in a different file because..."
-    ✅ "Have you considered this alternative approach?"
-    ✅ "This seems inconsistent with the pattern we established..."
-    ❌ Just implementing suggestions without evaluation

(rest of the file remains unchanged)
## Code Formatting and Testing

### Python Code Formatting
- **Black Version**: Always use `black==26.3.1` to match the version pinned in `requirements-dev.txt`
- **isort Configuration**: Use `isort --profile black` for import sorting to maintain compatibility with Black
- **Installation**: Use `pipx install black==26.3.1` and `pipx install isort`
- **Commands for formatting**:
  ```bash
  # Format with specific black version
  ~/.local/bin/black src/

  # Sort imports with black profile
  ~/.local/bin/isort --profile black src/

  # Check formatting (for CI compatibility)
  ~/.local/bin/black --check src/
  ~/.local/bin/isort --profile black --check-only src/
  ```

### Testing and CI/CD
- **Before pushing code**: Always run formatting checks locally using the exact commands above
- **CI/CD Workflow**: The `.github/workflows/ci-cd.yml` uses the same black version and isort profile
- **Docker Actions**: Use stable versions only:
  - `docker/login-action@v3`
  - `docker/setup-buildx-action@v3` 
  - `docker/metadata-action@v5`
  - `docker/build-push-action@v6`

## Subtitle System Implementation

### Complete Subtitle Functionality ✅

**Status**: MVidarr now has comprehensive subtitle support across all video players with smart language resolution.

#### Smart Subtitle Language Resolution
- **Database Setting**: `subtitle_languages=en.*` (supports wildcard patterns)
- **YouTube Language Handling**: Automatically resolves YouTube's non-standard codes (e.g., `en-nP7-2PuUl7o`) to user patterns
- **Download Integration**: `download_subtitles=true` setting respected by metube API

#### Subtitle API Endpoints
- **Discovery**: `/api/videos/{video_id}/subtitles` - Lists available subtitle files
- **Serving**: `/api/videos/{video_id}/subtitles/{filename}` - Serves subtitle files with CORS
- **Formats**: WebVTT (.vtt), SubRip (.srt), ASS (.ass), SSA (.ssa), MicroDVD (.sub)

#### Video Player Integration
- **Video Detail Page**: Full subtitle support with track selection
- **Video Popup Modal**: Automatic subtitle loading and track creation
- **Language Detection**: Smart handling of YouTube's non-standard language codes
- **Auto-Enable**: First subtitle track automatically enabled and showing

#### Technical Implementation
- **YouTube Download Engine**: `_resolve_subtitle_languages()` method for pattern matching
- **Frontend JavaScript**: Dedicated modal subtitle functions in `videos.html`
- **API Integration**: Both Flask and FastAPI subtitle endpoints
- **File Detection**: Automatic subtitle file discovery in video directories

#### User Experience
- Subtitles automatically available in all video players

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prefect421/mvidarr](https://github.com/prefect421/mvidarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
