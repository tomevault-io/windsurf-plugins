---
trigger: always_on
description: > **What this is:** A FastAPI web app that manages multiple groups of AI agents across any LLM tool. It's the unified control plane for monitoring agent observations, reviewing proposals, editing memory/prompts, and managing agent infrastructure — regardless of whether your agents use Claude Code, Codex, Gemini, Aider, Goose, or custom scripts.
---

# Agency — Agent Management Dashboard

> **What this is:** A FastAPI web app that manages multiple groups of AI agents across any LLM tool. It's the unified control plane for monitoring agent observations, reviewing proposals, editing memory/prompts, and managing agent infrastructure — regardless of whether your agents use Claude Code, Codex, Gemini, Aider, Goose, or custom scripts.

## Architecture

- **Framework:** FastAPI + Jinja2 + Tailwind CSS (CDN, no build step)
- **Database:** None — entirely filesystem-based. Reads markdown files with YAML frontmatter from agent directories.
- **Config:** `config.yaml` — defines agent groups, Agency settings. Written atomically (temp + rename).
- **Integrations:** Plugin system (`agency/integrations/`) translates between LLM tools and Agency's internal model. Each agent declares which integration it uses.
- **Dispatch:** Python-based scheduler (`agency/dispatch/run.py`) with platform-native timers (systemd on Linux, launchd on macOS).
- **Deployment:** User-level systemd service (`agency.service`) on port 8500.

## Project Structure

```
~/dev/agency/
├── agency/                    # Python package
│   ├── app.py                 # Main FastAPI app (~2500 lines)
│   ├── cli.py                 # CLI interface (agency inbox, status, decide, etc.)
│   ├── config.py              # Shared config utilities (normalize_agents, agent_names)
│   ├── __init__.py
│   ├── integrations/          # LLM integration plugin system
│   │   ├── __init__.py        # BaseIntegration, registry, config-driven loading
│   │   ├── integrations.yaml  # Which integrations are loaded (managed by admin UI)
│   │   ├── _template.py       # Scaffolding for new integrations
│   │   ├── agency/            # Official integrations
│   │   │   ├── claude_code.py
│   │   │   ├── codex.py
│   │   │   ├── gemini.py
│   │   │   ├── aider.py
│   │   │   ├── goose.py
│   │   │   ├── script.py
│   │   │   └── sdk.py
│   │   └── {author}/          # Community integrations
│   ├── dispatch/              # Dispatch system
│   │   ├── run.py             # Python dispatch runner (replaces dispatch.sh)
│   │   ├── install.py         # Platform-native timer installer
│   │   └── __init__.py
│   ├── workspaces/            # Workspace plugin system
│   │   ├── __init__.py        # BaseWorkspace, REGISTRY, migration
│   │   ├── tmux.py            # tmux session layout
│   │   ├── cursor.py          # Cursor IDE
│   │   ├── superset.py        # Superset.sh orchestrator
│   │   ├── ide.py             # Generic IDE (VS Code, Windsurf, JetBrains)
│   │   ├── chat.py            # Chat platforms (Slack, Mattermost, Discord)
│   │   └── custom.py          # Custom config file
│   └── templates/             # 27 Jinja2 templates
│       ├── base.html          # Layout: sidebar + main content
│       ├── home.html          # Mission control dashboard (fleet, pipeline, attention queue, activity)
│       ├── agents.html        # Agent list with health dots + integration badges
│       ├── agent_profile.html # Agent profile: identity, integration, timeline, schedule
│       ├── observations.html   # Observation list with filters
│       ├── observation_detail.html # Single observation + pipeline chain + status change
│       ├── proposals.html     # Proposal list
│       ├── proposal_detail.html # Proposal + pipeline chain + decide form
│       ├── decisions.html     # Decision list
│       ├── decision_detail.html # Single decision + pipeline chain
│       ├── documents.html     # Agent documents browser
│       ├── document_view.html # View/edit markdown, CSV, HTML
│       ├── logs.html          # Execution logs by date
│       ├── log_view.html      # Single log file
│       ├── prompts.html       # Dispatch prompts with agent assignments + schedule editing
│       ├── prompt_detail.html # View/edit prompt content
│       ├── memory.html        # Agent memory list
│       ├── memory_view.html   # View/edit memory
│       ├── admin.html         # Admin: redirects to settings
│       ├── admin_settings.html # Admin: app settings + installed integrations table
│       ├── admin_integrations.html # Admin: integration management + registration
│       ├── admin_dispatch.html # Admin: dispatch timer management
│       ├── admin_groups.html  # Admin: agent group list + management
│       ├── admin_org_edit.html # Create/edit org + dispatch schedule + default integration
│       ├── admin_agent_detail.html # Admin agent detail view
│       ├── setup.html         # First-run wizard
│       ├── setup_complete.html # Post-setup "touch grass" finale page
│       ├── workspaces.html        # Workspace list — runtime frontend configs
│       └── workspace_detail.html  # Workspace config file viewer/editor
├── tests/                     # Test suite (98 tests)
│   ├── conftest.py            # Shared fixtures
│   ├── test_integrations.py   # Registry, detection, base classes
│   ├── test_integration_claude_code.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christag/agency](https://github.com/christag/agency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
