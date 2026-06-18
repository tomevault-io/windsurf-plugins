---
trigger: always_on
description: Container-based development for isolated, reproducible environments. Use when running npm commands, installing packages, executing code, or managing project dependencies. Trigger phrases include "npm install", "run the build", "start the server", "install package", or any code execution request.
---


# Docker Development Skill

Execute all package installations and code execution inside Docker containers. This keeps the host machine clean and ensures consistent environments across projects.

---

## Behavioral Classification

**Type**: Autonomous Execution

**Directive**: EXECUTE, DON'T ASK

This skill enforces Docker-first development automatically. When you request npm/node operations, commands are executed inside Docker without asking for permission.

**Enforcement Behavior**:
- Blocks host-machine npm/node commands
- Suggests or transforms commands to use `docker exec`
- Automatically checks container status before operations

---

## Quick Start

For new projects, see **[setup.md](setup.md)** for the complete first-time setup guide.

**Minimal setup**:

```bash
# 1. Start container
docker compose --profile dev up -d

# 2. Verify running
docker ps --filter "name=my-project"

# 3. Run commands in container
docker exec my-project-dev-1 npm install
docker exec my-project-dev-1 npm test
docker exec my-project-dev-1 npm run build
```

---

## ENFORCEMENT RULES

### BLOCKED: Never Run on Host

**These commands are BLOCKED on the host machine:**

```
npm install, npm ci, npm run, npm test, npm exec
npx <anything>
yarn add, yarn install, yarn run
pnpm add, pnpm install, pnpm run
node <script>
tsx <script>
bun <script>
```

**Why?** Installing packages on the host:
- Pollutes the host machine with project-specific dependencies
- Creates version conflicts between projects
- Makes environments non-reproducible
- Can cause security issues with global packages

### REQUIRED: Use docker exec

**All Node.js commands MUST use this prefix:**

```bash
docker exec <container-name> <command>
```

**Note:** Use `docker exec` WITHOUT the `-it` flag in Claude Code:

```bash
# Correct (works in Claude Code)
docker exec my-project-dev-1 npm run build

# Wrong (fails with "not a TTY")
docker exec -it my-project-dev-1 npm run build
```

---

## Pre-Flight Check (MANDATORY)

**Before running ANY npm/node command, verify the container is running.**

```bash
docker ps --filter "name=<container-name>" --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

**Expected output:**
```
NAMES                   STATUS          PORTS
my-project-dev-1        Up X minutes    0.0.0.0:3000->3000/tcp
```

**If container is NOT running:**
```bash
cd /path/to/project
docker compose --profile dev up -d
docker ps --filter "name=<container-name>"
```

**If container shows "Exited":**
```bash
docker logs <container-name> --tail 20
docker compose --profile dev down
docker compose --profile dev up -d
```

---

## Quick Reference

### Check Container Status

```bash
docker ps --filter "name=<container-name>"
docker logs <container-name> --tail 50
curl -s http://localhost:<port> > /dev/null && echo "Running" || echo "Not running"
```

### Start/Stop Containers

```bash
docker compose --profile dev up -d        # Start
docker compose --profile dev down         # Stop
docker compose --profile dev restart dev  # Restart
docker compose --profile dev up -d --build  # Rebuild
```

### Execute Commands Inside Container

```bash
docker exec <container> npm install <package>     # Install package
docker exec <container> npm install -D <package>  # Install dev dependency
docker exec <container> npm test                  # Run tests
docker exec <container> npm run typecheck         # Type checking
docker exec <container> npm run lint              # Linting
docker exec <container> npm run build             # Build
docker exec <container> /bin/sh                   # Shell (use -it for interactive)
```

---

## When to Use docker exec

| Operation | Use docker exec? | Reason |
|-----------|------------------|--------|
| `npm install` | **Always** | Packages install in container only |
| `npm run dev` | No | Already running via docker-compose |
| `npm test` | Yes | Tests run in container environment |
| `npm run build` | Yes | Build happens in container |
| `git` commands | No | Git runs on host (manages files) |
| File editing | No | Volume mount syncs automatically |
| Database migrations | Yes | Uses container's Node environment |

---

## Container Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  HOST (macOS/Linux/Windows)                                 │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Docker Container (my-project-dev-1)                │   │
│  │                                                     │   │
│  │  Node 20 (Slim or Alpine)                           │   │
│  │  └── node_modules/ (container-only, NOT on host)    │   │
│  │  └── Dev server (port 3000)                         │   │
│  │                                                     │   │
│  │  Volume Mounts:                                     │   │
│  │  └── .:/app (source code sync)                      │   │
│  │  └── node_modules:/app/node_modules (persist deps)  │   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wrsmith108/docker-claude-skill](https://github.com/wrsmith108/docker-claude-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
