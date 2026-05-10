---
trigger: always_on
description: Instructions for any Claude Code session working *on* the open-forge plugin (not running it). Different audience from `plugins/open-forge/skills/open-forge/SKILL.md`, which is what an end-user's Claude reads to *use* the plugin.
---

# CLAUDE.md

Instructions for any Claude Code session working *on* the open-forge plugin (not running it). Different audience from `plugins/open-forge/skills/open-forge/SKILL.md`, which is what an end-user's Claude reads to *use* the plugin.

## What is open-forge

A Claude Code plugin/skill that turns "read a README, copy-paste 30 lines of bash, debug for hours" into a guided chat where Claude executes everything via the user's local CLI tools and the user only makes choices.

## Architecture — 3 layers, asked in 3 questions

A deployment is a tuple of three independent axes, asked in this order:

| # | Question | Layer | Examples |
|---|---|---|---|
| 1 | **What** to host? | software | OpenClaw, Ghost, Mastodon, Vaultwarden, Nextcloud |
| 2 | **Where** to host? | infra (cloud or local) | AWS / Hetzner / DigitalOcean / GCP / Azure / bring-your-own-VPS / **localhost** |
| 3 | **How** to host (within that cloud)? | infra-service + runtime | AWS: Lightsail blueprint, Lightsail Ubuntu + Docker, EC2 + native, EKS, ECS Fargate. Hetzner: Cloud CX + Docker, Cloud CX + native. localhost: Docker Desktop, native. |

The third question is *dynamically generated* from (software, cloud) — different clouds expose different compute services, and some software has vendor-bundled blueprints on specific clouds.

**Some infra services bundle the runtime** — EKS → Kubernetes, Lightsail OpenClaw blueprint → vendor's pre-baked install. In those cases the "runtime" question is not asked separately. **Other services give runtime choice** — EC2, plain VPS, localhost — there we ask Docker vs native vs k3s.

**Reusability is the test.** "Install Docker + run docker-compose" is the same on Lightsail Ubuntu, Hetzner CX-line, a DO droplet, and a localhost — write it once in the runtime layer, reference from every project. "Install k3s" is the same across clouds — write it once. Project recipes should be 80% software-specific concerns and contain *no* per-runtime install commands beyond a one-line link.

### File layout for the 3 layers

```
references/
├── projects/<sw>.md                       # software layer (thin)
├── infra/
│   ├── aws/
│   │   ├── lightsail-blueprint.md         # vendor-bundled, software-specific
│   │   ├── lightsail-ubuntu.md            # Lightsail as a plain VM
│   │   ├── ec2.md
│   │   ├── eks.md
│   │   └── ecs-fargate.md
│   ├── hetzner/cloud-cx.md
│   ├── digitalocean/droplet.md
│   ├── gcp/compute-engine.md
│   ├── byo-vps.md                         # user provides any Linux VPS, Claude SSH-es in
│   └── localhost.md                       # user's own machine, Claude runs commands directly
├── runtimes/
│   ├── docker.md                          # reusable wherever Docker works
│   ├── native.md                          # native installer (curl/apt)
│   └── kubernetes.md                      # reusable across EKS/GKE/AKS/k3s
└── modules/                               # cross-cutting (preflight, dns, tls, smtp providers, inbound forwarders, tunnels, backups, monitoring)
```

`localhost.md` is a first-class infra — for many projects (especially OpenClaw), running locally is the default upstream path. Same conversational UX as a cloud deploy; differences are: no SSH (Claude runs commands directly), no provisioning, public reach via tunnel (`references/modules/tunnels.md`).

## Operating principles

1. **Do more, ask less. Non-tech-friendly.** Default to autonomous execution. Only prompt the user for things only they can decide or provide: credentials, opinionated choices, things that touch their accounts at other companies. Hide everything Claude can figure out from the recipe.
2. **Towards production-ready architecture.** Even single-node hobby deploys should be on a path to backups, monitoring, TLS, key rotation, OS updates, and least-privilege firewalls. Don't write recipes that "work" but leave the system one outage away from data loss.
3. **Security in mind.** Treat tokens/keys as toxic — never log them, rotate after chat exposure, prefer fragment URLs over query strings. Default firewalls to closed; open ports explicitly. Default to SSH key auth; never password. Let's Encrypt for any public endpoint. Sandbox agent tool execution where the runtime supports it.
4. **One question at a time.** Use `AskUserQuestion` for structured choices. Reserve free-text for credentials and identifiers (domain names, emails). No upfront questionnaires.
5. **Auto-install with confirmation, never silently.** If `jq` or `aws` is missing, propose the install command, get one-line approval, then run.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhangqi444/open-forge](https://github.com/zhangqi444/open-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
