---
trigger: always_on
description: Airut is a self-hosted gateway that runs sandboxed Claude Code over email and
---

# CLAUDE.md - Operating Instructions for Airut

## Project Overview

Airut is a self-hosted gateway that runs sandboxed Claude Code over email and
Slack. Named "Airut" (Finnish: herald/messenger).

Network requests from containers are sandboxed; if requests fail, check
`.airut/network-allowlist.yaml`. See `doc/network-sandbox.md` for details.

## Documentation

High-level documentation in `doc/` (see `doc/README.md` for full list):

- `doc/architecture.md` — system architecture and data flow
- `doc/security.md` — security model (channel auth, isolation, credentials)
- `doc/execution-sandbox.md` — container isolation and resource limits
- `doc/network-sandbox.md` — network allowlist and proxy architecture
- `doc/ci-sandbox.md` — sandboxing GitHub Actions and CI pipelines
- `doc/deployment.md` — installation and server configuration
- `doc/github-app-setup.md` — GitHub App setup for agent identity
- `doc/email-setup.md` — email channel setup (IMAP/SMTP, DMARC)
- `doc/slack-setup.md` — Slack channel setup (Socket Mode, authorization rules)
- `doc/m365-oauth2.md` — Microsoft 365 OAuth2 setup for email (IMAP/SMTP)
- `doc/repo-onboarding.md` — onboarding new repositories
- `doc/gerrit-onboarding.md` — Gerrit-specific onboarding
- `doc/agentic-operation.md` — message-to-PR workflow patterns
- `doc/periodic-tasks.md` — cron-triggered scheduled tasks with email delivery
- `doc/testing.md` — test strategy, coverage enforcement, config migration
  testing

Implementation specs in `spec/` (see `spec/README.md` for full list):

- `spec/gateway-architecture.md` — core gateway design, channel abstraction,
  data flow, container execution
- `spec/authentication.md` — DMARC verification and sender authorization
- `spec/repo-config.md` — per-repo schema in server config
- `spec/declarative-config.md` — declarative config layer (schema metadata,
  migration, diffing, round-trip, config variables)
- `spec/config-reload.md` — live config reload (file watching, scope-aware
  application, deferred reload)
- `spec/multi-repo.md` — multi-repository support design
- `spec/slack-channel.md` — Slack channel implementation (Socket Mode)
- `spec/integration-tests.md` — end-to-end test specification
- `spec/periodic-tasks.md` — cron-triggered periodic tasks (scheduler,
  prompt/script modes, email delivery)
- `spec/sandbox.md` — sandbox library for safe containerized Claude Code
  execution
- `spec/sandbox-action.md` — reusable GitHub Action wrapping airut-sandbox
- `spec/sandbox-cli.md` — standalone sandbox CLI for CI and other environments
- `spec/image.md` — two-layer container image build strategy
- `spec/network-sandbox.md` — proxy lifecycle, resource scoping, log format,
  crash recovery
- `spec/masked-secrets.md` — scope-restricted credentials with proxy-level token
  replacement
- `spec/aws-sigv4-resigning.md` — AWS SigV4/SigV4A credential masking via proxy
  re-signing
- `spec/graphql-operation-allowlist.md` — generic GraphQL operation filtering at
  the proxy layer
- `spec/github-app-credential.md` — GitHub App credential support with
  proxy-managed token rotation
- `spec/dashboard.md` — web dashboard for task monitoring
- `spec/cli.md` — CLI subcommands, service management, self-update
- `spec/pr-workflow-tool.md` — PR workflow automation (ci.py, pr.py)
- `spec/local-ci-runner.md` — CI runner (single source of truth for all CI
  checks)

## Operational Workflows

The `workflows/` directory contains step-by-step guides. **When a user request
matches any pattern below, you MUST read the workflow file first before taking
any action.** This is not optional - the workflow contains critical steps and
context that cannot be summarized here.

- **Preparing a release** → `workflows/release.md`
  - Release notes, GitHub release creation (version derived from git tag)
- **Maintaining sandbox-action** → `workflows/sandbox-action.md`
  - Setup, changes, releases for `airutorg/sandbox-action` GitHub Action
- **Code review (automated)** → `workflows/code-review.md`
  - Project-specific review checklist used by the code review subagent

Read the entire workflow before starting. Workflows are living documents -
update them if outdated.

## Configuration

Repository-level configuration lives in `.airut/`. See `.airut/README.md` for
details.

Key files:

- `.airut/network-allowlist.yaml` — domains and URL prefixes the container can
  access
- `.airut/container/Dockerfile` — repo-defined container base image
- `.airut/sandbox.yaml` — sandbox-action configuration (CI only, not used by
  gateway)

All per-repo settings (model, effort, resource limits, credentials, container
environment) are configured in the server config (`~/.config/airut/airut.yaml`).
See `spec/repo-config.md` for the full schema.

## Security Model

- **Email authentication** — DMARC verification on trusted
  `Authentication-Results` headers
- **Sender authorization** — allowlist with exact addresses or `*@domain`
  wildcards
- **Container isolation** — each conversation has isolated workspace, Claude
  session, inbox/outbox
- **Network sandbox** — all HTTP(S) routed through mitmproxy enforcing allowlist
  from default branch
- **Credential isolation** — all secrets passed via environment variables, no
  host mounts

## CRITICAL: Workflow Tools


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airutorg/airut](https://github.com/airutorg/airut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
