---
trigger: always_on
description: This document is the onboarding guide for AI agents working on `nsys-ai`.
---

# 🤖 Agent Playbook — nsys-ai

This document is the onboarding guide for AI agents working on `nsys-ai`.

---

## Repository Structure

```
nsys-ai/
├── src/nsys_ai/          # Main Python package
│   ├── __main__.py        # CLI entry point (argparse-based)
│   ├── profile.py         # SQLite profile loader
│   ├── tui.py             # Tree TUI (curses)
│   ├── tui_timeline.py    # Timeline TUI (curses)
│   ├── tree.py            # NVTX tree data model
│   ├── search.py          # Kernel/NVTX search
│   ├── overlap.py         # Compute/NCCL overlap analysis
│   ├── summary.py         # Profile summary stats
│   ├── export.py          # HTML viewer export
│   ├── export_flat.py     # CSV/JSON flat export
│   ├── projection.py      # Time-range projection
│   ├── viewer.py          # Perfetto JSON export
│   ├── web.py             # Local HTTP server (stdlib HTTPServer + ThreadPool)
│   ├── skills/            # 🧩 Analysis skill system
│   │   ├── base.py        # Skill dataclass + execution
│   │   ├── registry.py    # Auto-discovery + lookup
│   │   └── builtins/      # 29 built-in analysis skills
│   ├── agent/             # 🤖 AI agent
│   │   ├── persona.py     # System prompt + identity
│   │   └── loop.py        # Analysis loop + skill selection
│   ├── ai/                # AI module (NVTX annotation, convergence)
│   └── templates/         # HTML templates (string.Template — no Jinja2)
├── tests/                 # pytest test suite
├── docs/                  # Documentation (8 guides + root causes)
│   ├── root-causes/       # 📖 Book of Root Causes
│   └── sqlite-explorer/   # Interactive schema explorer
├── agent/                 # Agent design docs (not code)
├── site/                  # GitHub Pages landing page
├── examples/              # Example profiles + download scripts
├── pyproject.toml         # Package config (setuptools)
├── ROADMAP.md             # Prioritized roadmap with issue links
└── .github/workflows/
    ├── ci.yml             # CI: Python 3.10/3.11/3.12
    ├── publish.yml        # PyPI publish on v* tag
    ├── pages.yml          # GitHub Pages on site/** changes
    ├── project-sync.yml   # Label → project board sync
    └── security.yml       # Security scanning
```

---

## Label System

### Priority Labels
| Label | Meaning |
|-------|---------|
| `P0-critical` | Must have — next sprint |
| `P1-high` | High priority — near term |
| `P2-medium` | Medium priority |
| `P3-low` | Nice to have — longer term |

### Pillar Labels
| Label | Meaning |
|-------|---------|
| `pillar/ai` | AI pillar (analysis, NLP, models) |
| `pillar/ui` | UI pillar (TUI, web, viewer) |

### Agent Workflow Labels (State Machine)

```
agent-ready → agent-in-progress → agent-review → (merged) → Done
                    ↕
              agent-blocked
```

| Label | Meaning | When to apply |
|-------|---------|---------------|
| `agent-ready` | Issue is fully spec'd, ready to pick up | Default for roadmap items |
| `agent-in-progress` | Agent is actively working on this | After claiming the issue |
| `agent-blocked` | Agent cannot proceed (needs info/dependency) | When stuck |
| `agent-review` | PR raised, awaiting human review | After `gh pr create` |

---

## Agent Workflow

### 1. Find an issue to work on

```bash
# List all agent-ready issues, highest priority first
gh issue list -R GindaChen/nsys-ai --label agent-ready --sort created
```

Pick the highest-priority issue (`P0` > `P1` > `P2` > `P3`).

### 2. Claim the issue

```bash
# Swap label: agent-ready → agent-in-progress
gh issue edit <NUM> -R GindaChen/nsys-ai \
  --remove-label "agent-ready" \
  --add-label "agent-in-progress"
```

### 3. Create a branch

```bash
git checkout main && git pull
git checkout -b feat/issue-<NUM>-<short-description>
```

Branch naming: `feat/issue-3-ask-command`, `fix/issue-9-multi-gpu`, etc.

### 4. Implement

- Code lives in `src/nsys_ai/`
- Add tests in `tests/`
- Follow existing code style (no formatter configured — match surrounding code)
- The package name is `nsys_ai` internally (Python module), `nsys-ai` externally (PyPI)

### 5. Test before pushing

```bash
# Install in dev mode
pip install -e '.[dev]'

# Smoke test CLI
python -m nsys_ai --help

# Run test suite
pytest tests/ -v --tb=short
```

All three must pass. CI runs on Python 3.10, 3.11, and 3.12.

### 6. Push and create PR

```bash
git push -u origin feat/issue-<NUM>-<short-description>

gh pr create -R GindaChen/nsys-ai \
  --title "<type>: <description>" \
  --body-file /tmp/pr_body.md
```

Then update the label:

```bash
gh issue edit <NUM> -R GindaChen/nsys-ai \
  --remove-label "agent-in-progress" \
  --add-label "agent-review"
```

### 6.1 Credential Safety (Mandatory)

- Never pass PR text inline with `gh pr create --body "..."`.
- Always write PR content to a file and use `--body-file` to avoid shell interpolation issues.
- Never include environment variable values, API keys, tokens, or command output that may contain secrets in PR titles, bodies, comments, commit messages, or logs.
- Never run commands that print secret-bearing values (for example key resolution or env dumps).
- If any sensitive text is exposed:
  1. Immediately sanitize/remove it from GitHub surfaces.
  2. Revoke/rotate affected credentials.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GindaChen/nsys-ai](https://github.com/GindaChen/nsys-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
