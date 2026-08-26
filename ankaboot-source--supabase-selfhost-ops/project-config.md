---
trigger: always_on
description: Deterministic, configuration-based Ansible deployer for self-hosted Supabase with optional components (Caddy SSO, monitoring, fail2ban, backups, UFW, LUKS).
---

# Ansible-Supabase — AI Context

## Purpose
Deterministic, configuration-based Ansible deployer for self-hosted Supabase with optional components (Caddy SSO, monitoring, fail2ban, backups, UFW, LUKS).

## Workflow
```
config.yml  ──>  setup.sh  ──>  env/supabase.yml  ──>  ansible-playbook
                                        │
                              playbook-supabase.yml
                              (regenerated from component toggles)
```

The user edits **only** `config.yml` (copy of `config.example.yml`). `setup.sh` validates, renders env vars into `env/supabase.yml`, regenerates `playbook-supabase.yml`, then runs `install.sh` (which bootstraps Ansible and executes the playbook).

## Key Files
| File | Purpose |
|------|---------|
| `config.example.yml` | User-facing template — add new config fields here |
| `config.yml` | Actual config (gitignored) — single source of truth |
| `setup.sh` | Orchestrator: validates, renders secrets, writes env vars |
| `env/supabase.yml` | Ansible vars file — rendered by `setup.sh`, consumed by playbook |
| `playbook-supabase.yml` | Generated playbook — roles enabled/disabled by component toggles |
| `install.sh` | Ansible runner (pip install + ansible-playbook) |

## Roles (`roles/`)
Each role follows Ansible convention:
- `tasks/main.yml` — idempotent deployment logic
- `defaults/main.yml` — default variables
- `templates/` — Jinja2 templates

### Role: docker
Installs Docker Engine (official APT repo), `docker-compose-plugin`, and adds `deploy_user` to the `docker` group. Always runs first as a prerequisite.

### Role: supabase
Clones the official Supabase repo, renders the Docker Compose stack, configures Kong (API gateway), sets up SSL certs (from Caddy or self-signed), and starts all Supabase services (Postgres, GoTrue, PostgREST, Realtime, Storage, Edge Functions, Studio, etc.). Six templates: `docker-compose-supabase.yml.j2`, `docker-compose-logs.yml.j2` (Logflare + Vector log-drain override, see below), `vector-logs.yml.j2` (<supabase_path>/volumes/logs/vector.yml — Vector pipeline that routes container logs to Logflare, container names templated with the `deploy_env` suffix), `kong-supabase.yml.j2`, `env-supabase.j2`, `start-supabase.sh.j2`.

The **log drain** (Studio dashboard logging) is a separate compose override: `docker-compose-logs.yml` (analytics/Logflare + vector). It ships in the repo always, but `start-supabase.sh` only boots it when `log_drain_enabled: true` (rendered by setup.sh from `required.enable_logging`, default `true` — set it `false` to disable log drain). The file is always rendered so `down` tears down orphaned analytics/vector when the drain is disabled.

### Role: caddy
Reverse proxy + automatic TLS + SSO. Four provider templates in `templates/`:
- `Caddyfile-github.j2` — uses `github_allow_list` (match sub)
- `Caddyfile-gitlab.j2` — uses `gitlab_allow_list` (match email)
- `Caddyfile-generic.j2` — uses `generic_allow_list` (match email)
- `Caddyfile-discord.j2` — no allow list; uses role-based auth via `admin_role_id` + `discord_guild_id`

Template selection: `{ src: "Caddyfile-{{ SSO_PROVIDER }}", dest: "/etc/caddy/Caddyfile" }`. Also creates the systemd unit and reloads/restarts caddy on config changes.

### Role: monitor
Deploys Grafana + Prometheus + Loki + cAdvisor + Node Exporter + Postgres Exporter + Promtail via Docker Compose. Ten templates including datasources, dashboards (pre-built server-stats dashboard), and config files. Supports anonymous access or basic auth with customizable passwords.

### Role: fail2ban
Installs fail2ban with a Postgres-specific jail that watches `/var/log/postgresql/postgresql.log` for failed auth attempts and bans offending IPs for 24 hours. Three templates: jail, action, and filter configs.

### Role: backup
Deploys pgBackRest running inside the Supabase `db` container (not a separate container), with continuous WAL archiving via native pgBackRest async mode, scheduled full + differential backups, point-in-time recovery (`restore.yml`), non-destructive restore verification (`restore-verify.yml`), on-demand backup (`backup.yml`), daily repo-integrity verification, Supabase Storage volume backup, pgsodium root-key backup, and Prometheus metrics + Grafana dashboard + alerting. Repo types: `minio` (local, default), `s3` (external), `posix` (local fs). Encryption is forced ON for external S3 repos. WAL archiving uses pgBackRest's native `archive-async` mode: the pgbackrest binary (Alpine) + shared libs are extracted from a Docker image to the host, a wrapper script sets `LD_LIBRARY_PATH` only for pgbackrest (not for PG itself), and `archive_command` runs `pgbackrest archive-push %p` directly. pgBackRest manages its own spool, queueing, and retry logic. All pgbackrest CLI commands (stanza-create, backup, verify, expire) execute via `docker exec supabase-db`. Templates: `pgbackrest.conf.j2`, `docker-compose-backup.yml.j2`, `backup-env.j2`, six `backup-scripts/*.j2`, `grafana/backup.json.j2`, `prometheus-backup-alerts.yml.j2`.

### Role: ufw

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ankaboot-source/supabase-selfhost-ops](https://github.com/ankaboot-source/supabase-selfhost-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
