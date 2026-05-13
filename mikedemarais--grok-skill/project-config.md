---
trigger: always_on
description: Guidelines for Claude Code when working on the **grok-skill** repository.
---

# CLAUDE.md — grok-skill

Guidelines for Claude Code when working on the **grok-skill** repository.

---

## Repository Overview

| Item | Value |
|------|-------|
| **Purpose** | Claude Code Skill for X/Twitter search via Grok 4 (OpenRouter) |
| **Location** | `~/.claude/skills/grok-skill/` (user skill directory) |
| **Dev Repo** | `~/git/grok-skill/` (this repo for development) |
| **Primary Script** | `scripts/grok.ts` (Bun TypeScript) |
| **API Provider** | OpenRouter (`x-ai/grok-4` model) |
| **API Key Source** | Environment variable `$OPENROUTER_API_KEY` |
| **Runtime** | Bun |

---

## File Structure

```
grok-skill/
├── .gitignore         # Protects secrets (.env, .env.*)
├── SKILL.md           # Skill definition for Claude Code
├── README.md          # User documentation
├── CLAUDE.md          # This file (development guidelines)
└── scripts/
    └── grok.ts        # Main executable TypeScript script
```

---

## Architecture

### Module Responsibilities

**CLI Parsing (`parseArgs`):**
- Multi-value flag collection for `--include` / `--exclude`
- Validates `--mode` (on|off|auto)
- Validates numeric bounds for `--max`, `--min-faves`, `--min-views`
- Enforces mutual exclusivity between include/exclude

**Normalization:**
- Strips '@' prefix from handles
- Lowercases and deduplicates handles
- Caps handle lists at 10
- Validates dates to real calendar values (rejects 2025-02-30)
- Clamps `--max` to [1..50] range

**API Client (`fetchWithRetry`):**
- 30-second timeout via AbortController
- Exponential backoff retries (up to 3 attempts)
- Honors `Retry-After` header for rate limits
- Retries on 408/429/5xx errors
- Logs request-id when available

**Request Building:**
- Constructs `extra_body.search_parameters` with:
  - `mode`, `return_citations`, `max_search_results`
  - `from_date`, `to_date` (ISO format)
  - `sources` array with X-specific filters
- Sets temperature: 0.2, max_tokens: 1200

**Response Handling:**
- Primary: `choices[0].message.content`
- Fallback: `choices[0].delta.content`
- Citations: Checks `citations`, `choices[0].message.citations`, `extra.citations`
- Fallback: Extracts `https://x.com/*` and `https://twitter.com/*` URLs from text

**Error Handling:**
- Exit code 2: Usage/validation errors
- Exit code 3: Network/timeout errors
- Exit code 4: JSON parse errors

---

## Development Workflow

### 1. Making Changes

When modifying the skill:

1. **Edit files in this repo** (`~/git/grok-skill/`)
2. **Test changes** here first (see Testing section below)
3. **Deploy to skill directory** when ready:
   ```bash
   rsync -av --delete ~/git/grok-skill/ ~/.claude/skills/grok-skill/ --exclude '.git'
   ```
4. **Restart Claude Code** to reload the skill

### 2. Testing

```bash
# Set API key (choose your preferred method)
export OPENROUTER_API_KEY="sk-or-..."

# Test minimal query
bun scripts/grok.ts --q "test query"

# Test with filters
bun scripts/grok.ts \
  --q "recent activity" \
  --include "@OpenAI" "@AnthropicAI" \
  --from 2025-11-01 \
  --to 2025-11-07 \
  --mode on \
  --max 8

# Test error handling
bun scripts/grok.ts --q "test" --from "2025-02-30"  # Should reject invalid date
bun scripts/grok.ts --q "test" --mode invalid       # Should reject invalid mode
unset OPENROUTER_API_KEY && bun scripts/grok.ts --q "test"  # Should show helpful error
```

### 3. Validation Checklist

Before deploying changes:
- [ ] Script runs without errors
- [ ] `OPENROUTER_API_KEY` is set in environment
- [ ] JSON output includes: `query`, `summary`, `citations`, `usage`, `model`
- [ ] SKILL.md frontmatter is valid YAML
- [ ] Script is executable (`chmod +x scripts/grok.ts`)
- [ ] All personal references removed (no machine-specific paths)
- [ ] No secrets in code or git history

---

## Key Implementation Details

### Script Arguments

| Flag | Type | Description | Default |
|------|------|-------------|---------|
| `--q` | string | Query text (required) | - |
| `--mode` | `on\|off\|auto` | Live Search mode | `auto` |
| `--include` | string[] | X handles to include (max 10) | `[]` |
| `--exclude` | string[] | X handles to exclude (max 10) | `[]` |
| `--from` | YYYY-MM-DD | Start date for search | - |
| `--to` | YYYY-MM-DD | End date for search | - |
| `--max` | number | Max search results (1-50) | `12` |
| `--min-faves` | number | Min favorites per tweet | - |
| `--min-views` | number | Min views per tweet | - |

**Constraints:**
- `--include` and `--exclude` are mutually exclusive
- Dates must be in ISO format (YYYY-MM-DD) and valid calendar dates
- Handles are automatically stripped of '@' prefix, lowercased, and deduplicated

### OpenRouter API Configuration

**Request Structure:**
```typescript
{
  model: "x-ai/grok-4",  // Configurable via GROK_MODEL env
  messages: [
    { role: "system", content: "You are Grok 4 answering with X/Twitter Live Search..." },
    { role: "user", content: query }
  ],
  extra_body: {
    search_parameters: {
      mode: "auto|on|off",
      return_citations: true,
      max_search_results: 12,
      from_date: "2025-11-01",
      to_date: "2025-11-07",
      sources: [{
        type: "x",
        included_x_handles: ["handle1", "handle2"],
        post_favorite_count: 50,
        post_view_count: 0
      }]
    }
  },
  temperature: 0.2,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikedemarais/grok-skill](https://github.com/mikedemarais/grok-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
