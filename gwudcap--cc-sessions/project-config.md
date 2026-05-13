---
trigger: always_on
description: A Claude Code workflow framework that enforces structured AI pair programming through Discussion-Alignment-Implementation-Check (DAIC) methodology.
---

# cc-sessions

A Claude Code workflow framework that enforces structured AI pair programming through Discussion-Alignment-Implementation-Check (DAIC) methodology.

## What It Does

cc-sessions transforms Claude Code into a disciplined workflow system where Claude must discuss approaches before writing code, maintains persistent task context across sessions, and provides specialized agents for complex operations. Available in both Python and JavaScript with complete feature parity.

## Core Concepts

### DAIC Enforcement
- **Discussion Mode**: Default state. Edit/Write/MultiEdit tools blocked until explicit approval
- **Implementation Mode**: Activated by trigger phrases ("yert", "make it so", "run that")
- **Todo-Based Boundaries**: Approved TodoWrite lists define exact implementation scope
- **Todo Change Detection**: Blocks unauthorized todo modifications with detailed diff and required "SHAME RITUAL" response format
- **Automatic Return**: Returns to discussion mode when all todos complete
- **Configurable**: Customize trigger phrases, blocked tools, and enforcement patterns

### Task Management
- **Priority Prefixes**: h- (high), m- (medium), l- (low), ?- (investigate)
- **Two Task Types**: File-based for focused work, directory-based for multi-phase projects
- **Branch Enforcement**: Automatic git branch creation and validation (optional, can be disabled)
- **Persistent Context**: Task state preserved across session restarts
- **Directory Tasks**: Multi-subtask workflows on a single feature branch

### Sessions State System
Unified state management in `sessions/sessions-state.json`:
- `current_task` - Active task with frontmatter integration
- `mode` - Current DAIC mode (discussion/implementation)
- `active_protocol` - Currently running protocol (CREATE/START/COMPLETE/COMPACT/None)
- `todos` - Active and stashed todo lists
- `flags` - Context warnings, subagent status
- `metadata` - Runtime state (kickstart progress, update checks)
- `api` - Protocol-specific command permissions

### Sessions Config System
User preferences in `sessions/sessions-config.json`:
- **Environment**: developer_name, os, shell
- **Trigger Phrases**: Customizable for all mode transitions
- **Git Preferences**: Branch naming, commit styles, auto-merge/push, submodules
- **Feature Toggles**: branch_enforcement, task_detection, auto_ultrathink, icon_style, context warnings
- **Blocking Patterns**: implementation_only_tools, bash_read_patterns, bash_write_patterns

### Templated Protocols
Configuration-driven protocol system that auto-adapts based on user preferences:
- **Template Variables**: `{default_branch}`, `{submodules_field}`, `{submodule_context}`, etc.
- **Conditional Sections**: Entire protocol chunks appear/disappear based on configuration
- **Auto-Loading**: `load_protocol_file()` helper eliminates manual file reading
- **Four Main Protocols**: task-creation, task-startup, task-completion, context-compaction
- **Kickstart Onboarding**: Interactive first-run tutorial system

## Key Files

### Core State & Configuration
- `cc_sessions/hooks/shared_state.py|.js` - State/config management, enums (IconStyle, etc.), atomic file operations, directory task helpers
- `sessions/sessions-state.json` - Unified runtime state (git-ignored)
- `sessions/sessions-config.json` - User preferences and customization (includes icon_style enum)

### Hook System
- `cc_sessions/hooks/sessions_enforce.py|.js` - Pre-tool DAIC enforcement with command analysis
- `cc_sessions/hooks/sessions_enforce.py:319-376|.js:397-454` - Todo change blocking with diff display and "SHAME RITUAL" format
- `cc_sessions/hooks/session_start.py|.js` - Session initialization and update detection
- `cc_sessions/hooks/user_messages.py|.js` - Trigger phrase detection and protocol loading
- `cc_sessions/hooks/post_tool_use.py|.js` - Todo completion detection and mode transitions
- `cc_sessions/hooks/subagent_hooks.py|.js` - Subagent context protection
- `cc_sessions/hooks/kickstart_session_start.py|.js` - Onboarding protocol loader

### Sessions API
- `cc_sessions/scripts/api/__main__.py` or `index.js` - API entry point
- `cc_sessions/scripts/api/router.py|.js` - Command routing with subsystem delegation
- `cc_sessions/scripts/api/state_commands.py|.js` - State inspection and update management
- `cc_sessions/scripts/api/config_commands.py|.js` - Configuration management with feature toggles
- `cc_sessions/scripts/api/task_commands.py|.js` - Task operations and index support
- `cc_sessions/scripts/api/protocol_commands.py|.js` - Protocol-specific commands
- `cc_sessions/scripts/api/kickstart_commands.py|.js` - Onboarding flow management

### Protocols
- `cc_sessions/protocols/task-creation/` - Task creation with directory structure confirmation
- `cc_sessions/protocols/task-startup/` - Task initialization with conditional guidance
- `cc_sessions/protocols/task-completion/` - Completion workflow with commit templating
- `cc_sessions/protocols/context-compaction.md` - Mid-task context management
- `cc_sessions/protocols/kickstart/` - Interactive onboarding system (11 protocols)

### Specialized Agents
- `cc_sessions/agents/context-gathering.md` - Creates task context manifests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GWUDCAP/cc-sessions](https://github.com/GWUDCAP/cc-sessions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
