---
trigger: always_on
description: **EVERYTHING is managed in this repository via code:**
---

# Homeserver Infrastructure Management Instructions

## IMPORTANT: Infrastructure as Code Philosophy

**EVERYTHING is managed in this repository via code:**

- **Terraform**: Network configuration (UniFi), Proxmox VMs/containers
- **Ansible**: Host configuration, container setup
- **ArgoCD**: Kubernetes applications via applicationsets (ONLY for main K3s
  cluster at 10.11.12.11)
- **NO manual commands** on servers
- **NO manually created applications** - everything is defined in code
- **NO backwards compatibility** - we fix things properly, not with workarounds

**CRITICAL REMINDERS:**

- **NEVER run SSH commands to fix things** - Always update Ansible/Terraform
  instead
- **NEVER delete files/directories without listing contents first** - Use
  `ls -laR` before any `rm`
- **NEVER commit without testing locally first** - Build/test before pushing
- **Slow down and be thorough** - Check all related files, read project docs
  first

## Bash Guidelines

**IMPORTANT: Avoid commands that cause output buffering issues:**

- DO NOT pipe output through `head`, `tail`, `less`, or `more` when monitoring
  or checking command output
- DO NOT use `| head -n X` or `| tail -n X` to truncate output - these cause
  buffering problems
- Instead, let commands complete fully, or use `--max-lines` flags if the
  command supports them
- For log monitoring, prefer reading files directly rather than piping through
  filters

**When checking command output:**

- Run commands directly without pipes when possible
- If you need to limit output, use command-specific flags (e.g., `git log -n 10`
  instead of `git log | head -10`)
- Avoid chained pipes that can cause output to buffer indefinitely

## Related Source Code

- **OpenClaw**: `../openclaw` - Personal AI assistant source code

### OpenClaw Development Guidelines

When modifying openclaw source code at `../openclaw`:

1. **Read `../openclaw/CLAUDE.md` first** - Contains project-specific guidelines
2. **Use pnpm, not npm** - Install deps with `pnpm install`, build with
   `pnpm build`
3. **Test before committing** - Run `pnpm lint && pnpm build && pnpm test`
4. **Use the committer script** - `scripts/committer "<msg>" <file...>` instead
   of manual git add/commit
5. **TypeScript types have TWO locations**:
   - Type definitions in `src/config/types.*.ts`
   - Zod validation schemas in `src/config/zod-schema.ts`
   - **BOTH must be updated** when adding new config options
6. **Install node_modules locally** - Required to run tests before committing

## Service Location Map

### Proxmox Infrastructure

**Proxmox Host (10.11.12.10)**: Main hypervisor running containers and VMs

**Containers (LXC):**

- **CT 100** (stopped): homeassistant
- **CT 101** (stopped): adguard
- **CT 103** (stopped): frigate
- **CT 104** (10.11.12.104): nginxproxymanager - Reverse proxy
- **CT 111**: homepage - Dashboard
- **CT 112** (10.11.12.140): webapp - Web App on port 2368 (proxied via
  nginx on port 80)
- **CT 117** (10.11.12.35): openclaw - Personal AI assistant on port 18789
- **CT 121** (10.11.12.30): circleci-runner - CircleCI self-hosted runners
- **CT 200** (10.11.12.11): k3s-cluster - Main Kubernetes cluster

**Virtual Machines (KVM):**

- _(none currently running - Home Assistant migrated to K3s)_

**External Services (not in Proxmox):**

- **Raspberry Pi** (10.11.12.22): AdGuard Home (port 80) and Zigbee2MQTT
  (port 8080)
- **UniFi UDM** (10.11.12.1): Network controller on port 443

## Container Details and Log Locations

| Container          | IP         | Service      | Log Location                                                        | Notes                                |
| ------------------ | ---------- | ------------ | ------------------------------------------------------------------- | ------------------------------------ |
| CT 104 npm         | 10.11.12.104 | NPM          | `/data/logs/`                                                       | Nginx Proxy Manager                  |
| CT 112 webapp      | 10.11.12.140 | Web App      | `/var/log/webapp/`, `/var/log/nginx`                                | Web app on port 2368, nginx proxy on 80 |
| CT 121 circleci    | 10.11.12.30  | CircleCI K3s | `journalctl -u k3s`                                                 | Self-hosted runners cluster          |
| CT 200 k3s-cluster | 10.11.12.11  | Main K3s     | `kubectl logs` or Loki                                              | ArgoCD-managed services              |
| Raspberry Pi       | 10.11.12.22  | AdGuard/Z2M  | `/opt/AdGuardHome/data/querylog.json`, `/opt/zigbee2mqtt/data/log/` | Backup DNS + Zigbee                  |

**Quick log access examples:**

```bash
# Web App logs
ssh root@10.11.12.140 "tail -100 /var/log/webapp/webapp.log"

# K3s pod logs (from k3s-cluster container)
ssh root@10.11.12.11 "kubectl logs -n <namespace> <pod-name>"
```

## K3s Clusters

**IMPORTANT: There are TWO separate K3s clusters:**

1. **Main K3s Cluster (10.11.12.11)**: Runs home services, managed by ArgoCD
   - Uses the `k3s/` directory structure below
   - Deployed via ArgoCD ApplicationSets
   - Hosts: AdGuard Home, Traefik, External Secrets, etc.

2. **CircleCI K3s Cluster (10.11.12.30)**: Runs CircleCI self-hosted runners
   - NOT managed by ArgoCD

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ndbroadbent/homeserver-terraform-ansible-public](https://github.com/ndbroadbent/homeserver-terraform-ansible-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
