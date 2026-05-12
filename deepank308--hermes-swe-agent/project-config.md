---
trigger: always_on
description: AI Agent Infrastructure — Remote Claude Code dev environments triggered by Linear tickets.
---

# CLAUDE.md

AI Agent Infrastructure — Remote Claude Code dev environments triggered by Linear tickets.

## Architecture

```
Linear webhook ──HTTPS──▶ Cloudflare Tunnel ──▶ Orchestrator (t4g.nano)
                                                  │
                                         VPC private IP
                                                  │
                                    ┌─────────────▼─────────────┐
                                    │  Agent EC2 (m6i.xlarge)   │
                                    │  ┌──────────────────────┐ │
                                    │  │ agent-service :3000  │ │
                                    │  │ Claude Code CLI      │ │
                                    │  │ Docker (app stack)   │ │
                                    │  └──────────────────────┘ │
                                    └───────────────────────────┘
```

```
hermes-swe/
├── ami/                  # EC2 provisioning & firewall scripts
│   ├── setup.sh          # Base AMI deps: Docker, Node 24, pnpm, yarn, gh, Claude Code
│   ├── prepare-ami.sh    # Pre-bake AMI: clone repos, delegate to repo scripts
│   ├── init-instance.sh  # Boot-time init: pull code, delegate to repo scripts
│   ├── firewall.sh       # Outbound allowlist (ipset/iptables)
│   ├── allowed-domains.txt  # Base firewall domains (shared across all repos)
│   ├── preview-launch.sh # Generic: reads launch.json, runs preview, creates Cloudflare tunnel
│   ├── <name>/           # Repo-specific scripts (scriptsDir: "ami/<name>")
│   │   ├── setup.sh      # Yarn 1.22.22, Turbo, kernel tuning, Docker pulls
│   │   ├── prepare.sh    # Build app, pre-commit, Docker compose build
│   │   ├── pre-agent.sh  # Start db16 early (MCP servers need DB)
│   │   ├── post-agent.sh # Docker services, yarn build
│   │   ├── launch.json   # Preview configurations (copied to workspace/.claude/)
│   │   ├── preview/      # Preview scripts (copied to workspace/.claude/preview/)
│   │   └── allowed-domains.txt  # App-specific firewall domains
│   └── hermes-swe/       # hermes-swe scripts (scriptsDir: "ami/hermes-swe")
│       └── post-agent.sh # Auto-detect package manager, install deps
├── agent-service/        # Runs ON agent EC2 — manages Claude sessions
│   ├── src/index.ts      # HTTP server (/run, /message, /stop, /health)
│   ├── src/session.ts    # Claude session lifecycle
│   └── prompts/system.md # System prompt — routes tickets to workflow skills
├── orchestrator/         # Runs ON orchestrator EC2 — webhook + EC2 lifecycle
│   ├── src/index.ts      # Fastify server, webhook routes, OAuth
│   ├── src/webhook-handler.ts  # Linear webhook → provision → run → teardown
│   ├── src/ec2.ts        # Launch/terminate EC2 instances
│   ├── src/session-store.ts    # S3-backed session state (cached in memory)
│   └── src/slack.ts      # Threaded Slack notifications
├── skills/               # Workflow skills — copied to ~/.claude/skills/ at boot
│   ├── full-development/ # Features, enhancements, refactors (plan → TDD → PR)
│   ├── debugging/        # Bug reports, Sentry issues (investigate → fix → PR)
│   └── simple-question/  # Questions, clarifications (research → answer)
├── scripts/
│   ├── aws-setup.sh      # Create AWS resources (SGs, S3, Secrets Manager)
│   ├── bake-ami.sh       # Automated AMI baking (--repo flag, run from orchestrator)
│   ├── setup-orchestrator.sh   # Fresh orchestrator EC2 setup
│   └── deploy-orchestrator.sh  # Deploy updates to orchestrator
├── repos.json            # Repo configs (amiName, scriptsDir, workspaceDir, secrets, instanceType)
└── package.json          # Root (pnpm workspace)
```

## Flow

1. Linear ticket assigned to agent → webhook fires
2. Orchestrator receives webhook via Cloudflare Tunnel
3. Orchestrator launches agent EC2 from pre-baked AMI
4. Agent EC2 boots: pulls code, starts Docker stack, starts agent-service
5. Orchestrator sends `/run` to agent-service with prompt
6. Claude works on the ticket (reads code, runs tests, creates PR)
7. Agent-service reports progress to Linear (thoughts, actions, completion)
8. On completion, agent-service calls orchestrator `/callback`
9. Orchestrator terminates EC2, notifies Slack, processes queue

## Network & Security

### Security Groups

| Security Group             | Inbound Rules                  | Access Method                            |
| -------------------------- | ------------------------------ | ---------------------------------------- |
| `hermes-orchestrator-sg`   | Port 3002 from agent SG        | SSH: key pair                            |
|                            |                                | Webhooks: Cloudflare Tunnel (bypasses SG)|
| `hermes-agent-sg` (agents) | Port 22 from orchestrator SG   | SSH: hop through orchestrator            |
|                            | Port 3000 from orchestrator SG | API: orchestrator only                   |

### Traffic Paths

| Path                     | Route                                      | Auth                                        |
| ------------------------ | ------------------------------------------ | ------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Deepank308/hermes-swe-agent](https://github.com/Deepank308/hermes-swe-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
