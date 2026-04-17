---
trigger: always_on
description: **autorun works identically in both Claude Code and Gemini CLI**, providing unified safety features, commands, and autonomous execution capabilities across both platforms.
---

# Autorun Workspace - Gemini CLI

**autorun works identically in both Claude Code and Gemini CLI**, providing unified safety features, commands, and autonomous execution capabilities across both platforms.

**For Claude Code:** See [CLAUDE.md](CLAUDE.md) for Claude Code-specific installation (uses `claude plugin install`).

## Installation (Gemini CLI)

### From GitHub (Production - Recommended)

```bash
# Install directly via Gemini extension system
gemini extensions install https://github.com/ahundt/autorun.git

# Verify
gemini extensions list  # Should show: ar@0.10.1, pdf-extractor@0.10.1
```

### From Local Clone (Development)

```bash
git clone https://github.com/ahundt/autorun.git && cd autorun

# Option 1: UV (recommended - faster, better dependency management)
uv run python -m plugins.autorun.src.autorun.install --install --force

# Option 2: pip fallback (if UV not available)
pip install -e . && python -m plugins.autorun.src.autorun.install --install --force

# REQUIRED: Install as UV tool for global CLI availability (works with both Gemini and Claude)
# This makes 'autorun' and 'claude-session-tools' commands globally available
# which are needed for proper daemon operation and session management
# Useful for: autorun --restart-daemon, autorun --install, autorun --status, etc.
cd plugins/autorun && uv tool install --force --editable .

# Verify installation
gemini extensions list    # Should show: ar@0.10.1, pdf-extractor@0.10.1
autorun --status        # Verifies UV tool installation works
```

**Install UV (if needed):**
```bash
# macOS/Linux:
curl -LsSf https://astral.sh/uv/install.sh | sh

# Homebrew:
brew install uv

# Windows:
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Test Installation

```bash
# Start new Gemini session
gemini

# Test
/ar:st  # Expected: "AutoFile policy: allow-all"
```

### Gemini-Specific Configuration

**IMPORTANT**: Hooks require explicit enablement in Gemini CLI settings (not required for Claude Code).

Edit `~/.gemini/settings.json` and add:

```json
{
  "tools": {
    "enableHooks": true,
    "enableMessageBusIntegration": true
  }
}
```

**Why Required**: Without these settings, autorun hooks will not execute in Gemini CLI. The safety features (command blocking, file policies) depend on hooks.

**Version Requirement**: Gemini CLI v0.28.0 or later recommended.

Update Gemini CLI:
```bash
# Using Bun (faster)
bun install -g @google/gemini-cli@latest

# Or using npm
npm install -g @google/gemini-cli@latest
```

Verify version:
```bash
gemini --version  # Should show 0.28.0 or later
```

## Autorun Integration Benefits

### Safety Features (Works in Both CLIs)

- **File Policies**: Control file creation (`/ar:a`, `/ar:j`, `/ar:f`)
- **Command Blocking**: Prevent dangerous operations (rm, git reset --hard, etc.)
- **Plan Export**: Auto-save plans to notes/ directory
- **Three-Stage Verification**: Ensures thorough task completion
- **Task Tracking**: Monitor task completion across sessions

**Commands work identically in both CLIs:**

```bash
/ar:st              # Show current AutoFile policy
/ar:a               # Allow all file creation
/ar:j               # Justify new files
/ar:f               # Find and modify existing files only (strictest)
/ar:go <task>       # Start autonomous execution
/ar:sos             # Emergency stop
/ar:tasks           # Toggle task staleness reminders on/off or set threshold
/ar:task-status     # Show task lifecycle status and incomplete tasks
/ar:pn              # Create new structured plan
/ar:pr              # Refine existing plan
/ar:pe              # Show plan export status
/ar:no <pattern>    # Block command pattern in session
/ar:ok <pattern>    # Allow blocked command in session
```

See [README.md](README.md) for the complete command reference (77 slash commands).

#### Task Staleness Reminders (v0.9)

Task staleness reminders work identically in both CLIs. When 25+ tool calls pass without TaskCreate/TaskUpdate, autorun injects a reminder. Use `/ar:tasks` to configure.

### Gemini Vision + Autorun Safety

**Use Gemini's superior vision capabilities with autorun's safety guards active:**

#### Image Analysis with File Policy Control

```bash
# Set strict policy: only modify existing files
/ar:f

# Analyze UI mockup and generate code (respects policy)
gemini -i notes/ui_mockup.png -c "Convert this mockup to React components"

# Autorun ensures:
# - No new files created (policy: SEARCH mode)
# - Suggests modifying existing components
# - Blocks dangerous bash commands
```

#### Architecture Diagram Analysis

```bash
# Analyze architecture diagram and implement
gemini -i notes/architecture.png -c "Implement the service layer shown in this diagram"

# Autorun provides:
# - File creation control via /ar:j (requires justification)
# - Command blocking (prevents accidental destructive operations)
# - Plan export (auto-saves implementation plan)
```

#### Video & Audio Analysis

Gemini supports video (1 FPS sampling) and audio (transcription + diarization) with autorun safety:

```bash
# Video UX analysis with safety
gemini -m gemini-3-pro-preview "Analyze notes/demo_video.mp4 for UX issues"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahundt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
