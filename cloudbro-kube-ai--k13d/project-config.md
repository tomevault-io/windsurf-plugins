---
trigger: always_on
description: default_agent: "@dev-agent"
---

# AGENTS.md
version: 1
default_agent: "@dev-agent"

> **Consulted files (agent must populate before committing):**
>
> **README.md files:**
> - `/README.md` - Main project README with features and getting started
> - `/docs-site/docs/user-guide/tui.md` - TUI guide with navigation and shortcuts
> - `/docs-site/docs/user-guide/web.md` - Web UI guide
> - `/docs-site/docs/getting-started/configuration.md` - Configuration guide
> - `/docs-site/docs/concepts/mcp-integration.md` - MCP server architecture and integration
>
> **Documentation files:**
> - `/docs-site/docs/getting-started/installation.md` - Installation overview
> - `/docs-site/docs/deployment/docker.md` - Docker and Docker Compose setup
> - `/docs-site/docs/deployment/kubernetes.md` - Kubernetes deployment
> - `/docs-site/docs/concepts/architecture.md` - System architecture overview
> - `/docs-site/docs/ai-llm/benchmarks.md` - AI model benchmark notes
> - `/CONTRIBUTING.md` - Contribution guidelines
> - `/SECURITY.md` - Security policy
>
> **Build/Config files:**
> - `/go.mod` - Go version (1.25.8) and dependencies
> - `/Makefile` - Build, test, and deployment commands
> - `/.goreleaser.yaml` - Release configuration
> - `/.github/workflows/ci.yml` - CI/CD workflow

---

## Project Overview

**k13d (kube-ai-dashboard-cli)** is a comprehensive Kubernetes management tool that provides both a terminal-based UI (TUI) and a web-based dashboard with an integrated agentic AI assistant. It combines the TUI experience of [k9s](https://k9scli.io/) with the AI-powered intelligence of [kubectl-ai](https://github.com/GoogleCloudPlatform/kubectl-ai).

### Core Value Proposition
- **Dual Interface**: Both TUI (terminal) and Web UI with same feature set
- **k9s Parity**: Full-featured TUI dashboard with Vim-style navigation
- **kubectl-ai Intelligence**: Agentic AI loop with tool-use (Kubectl, Bash, MCP)
- **Deep Synergy**: AI analysis with full context (YAML + Events + Logs)
- **Enterprise Ready**: Authentication, audit logging, and report generation

---

## Agent Persona and Scope

- **@dev-agent** - pragmatic, conservative, test-first, risk-averse.
- **Scope:** propose, validate, and prepare code/docs patches; run local build/test commands; create PR drafts.
- **Not allowed:** push images/releases, modify CI or infra, or merge without human approval.

---

## Explicit Non-Goals

The agent should NOT do the following unless explicitly requested:
- Propose refactors without a clear bug, performance, or maintenance justification
- Change public APIs without explicit request
- Reformat unrelated code
- Rename files or symbols for stylistic reasons
- Introduce new dependencies unless required to fix a bug or implement a requested feature

---

## Tech Stack and Environment

### Languages & Frameworks
- **Language:** Go 1.25.8+
- **TUI Framework:** [tview](https://github.com/rivo/tview) with [tcell](https://github.com/gdamore/tcell/v2)
- **Web Framework:** Standard library `net/http` with embedded static files
- **AI Integration:** Custom OpenAI-compatible HTTP client (supports OpenAI, Ollama, Anthropic)
- **MCP Integration:** JSON-RPC 2.0 stdio protocol for tool extensibility
- **Kubernetes Client:** client-go v0.35.2, metrics v0.35.2
- **Database:** CGO-free SQLite (modernc.org/sqlite) for audit logs and settings
- **Authentication:** Session-based with bcrypt password hashing (SHA256 legacy fallback)

### Key Dependencies
```
github.com/rivo/tview v0.42.0           # TUI framework
github.com/gdamore/tcell/v2 v2.13.8     # Terminal cell handling
github.com/GoogleCloudPlatform/kubectl-ai # AI agent integration
github.com/adrg/xdg v0.5.3               # XDG directory support
modernc.org/sqlite v1.46.1              # CGO-free SQLite
k8s.io/client-go v0.35.2                # Kubernetes client
k8s.io/metrics v0.35.2                  # Metrics API
```

### Skills (Pattern Reference Documents)

| Skill | File | Key Patterns |
|-------|------|--------------|
| k9s Patterns | `skills/k9s-patterns.md` | MVC architecture, Action system, Plugin/HotKey, Skin, XDG config |
| kubectl-ai Patterns | `skills/kubectl-ai-patterns.md` | Agent Loop, Tool System, LLM abstraction, MCP integration |
| Headlamp Patterns | `skills/headlamp-patterns.md` | Plugin Registry, Multi-Cluster, Response Cache, OIDC, i18n |
| K8s Dashboard Patterns | `skills/kubernetes-dashboard-patterns.md` | DataSelector, Multi-Module, Request-Scoped Client |

**Usage Guide**: `skills/README.md`

---

## Repository Structure

```
k13d/
├── cmd/
│   ├── kube-ai-dashboard-cli/main.go   # Main entry point (TUI + Web modes)
│   ├── bench/main.go                   # Benchmark tool
│   └── eval/main.go                    # Evaluation tool
├── pkg/
│   ├── ui/                             # TUI components
│   │   ├── app.go                      # Main application
│   │   ├── app_*.go                    # App lifecycle & callbacks
│   │   ├── dashboard.go                # Resource dashboard view
│   │   ├── assistant.go                # AI assistant panel
│   │   └── resources/                  # Resource-specific views
│   ├── web/                            # Web UI server
│   │   ├── server.go                   # HTTP server & API handlers
│   │   ├── auth.go                     # Authentication system

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudbro-kube-ai/k13d](https://github.com/cloudbro-kube-ai/k13d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
