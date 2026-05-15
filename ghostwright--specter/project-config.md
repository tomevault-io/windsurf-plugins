---
trigger: always_on
description: Go CLI that deploys AI agent VMs on Hetzner Cloud in 90 seconds. Interactive TUI (Bubbletea v2). Automatic DNS (Cloudflare) and TLS (Let's Encrypt via Caddy). Apache 2.0.
---

# Specter

Go CLI that deploys AI agent VMs on Hetzner Cloud in 90 seconds. Interactive TUI (Bubbletea v2). Automatic DNS (Cloudflare) and TLS (Let's Encrypt via Caddy). Apache 2.0.

## Build

```bash
make build          # -> bin/specter
make lint           # go vet + gofmt check
make test           # go test ./...
make ci             # lint + build (same as CI workflow)
make install        # go install to $GOPATH/bin
make clean          # rm -rf bin/ dist/
```

## Project Structure

```
cmd/specter/
  main.go                       Entry point, Cobra root
  commands/
    root.go                     Root command, global flags
    deploy.go                   VM creation + DNS + SSH provisioning + TLS
    destroy.go                  VM + DNS teardown
    init.go                     Setup wizard (tokens, firewall, server cache)
    image.go                    Golden snapshot build + list
    list.go                     Agent inventory with health checks
    status.go                   Single agent status
    ssh.go                      SSH into VM
    logs.go                     systemd journal streaming
    update.go                   Agent restart
    version.go                  Version output

internal/
  cloudflare/client.go          Cloudflare DNS API (raw HTTP, not SDK)
  config/
    config.go                   ~/.specter/config.yaml management
    server_types.go             Hetzner server type cache + fuzzy match
    state.go                    Agent state persistence
  hetzner/client.go             Hetzner Cloud API (hcloud-go v2)
  templates/
    cloudinit.go                Cloud-init user-data template  [FROZEN]
    systemd.go                  systemd unit template          [FROZEN]
    caddyfile.go                Caddy reverse proxy template   [FROZEN]
  tui/
    app.go                      Main Bubbletea model (~1,300 lines)
    agent_list.go               Dashboard list view
    agent_detail.go             Agent detail panel
    deploy_form.go              Deploy form (huh v2)
    deploy_model.go             Deploy data types
    deploy_progress.go          Deploy phase progress
    image_build.go              Image build progress
    setup_wizard.go             First-run setup
    logs_viewport.go            Log viewer
    confirm_dialog.go           Confirmation dialogs
    help_overlay.go             Keyboard help
    status_bar.go               Bottom bar
    dashboard_styles.go         Lipgloss styles
    messages.go                 Bubbletea messages
    theme.go                    Color palette

pkg/version/version.go          Version vars (set via ldflags)
```

## Frozen Files - DO NOT MODIFY

These files are rigorously validated. Modifying them without 3 full deploy-test-destroy cycles will break production deploys.

| File | What Went Wrong Last Time |
|------|--------------------------|
| `internal/templates/systemd.go` | `ReadWritePaths` listed subdirectories instead of `/home/specter/app`. Bun couldn't write lockfiles. 100% deploy failure rate. |
| `internal/templates/cloudinit.go` | Had `systemctl restart caddy` in runcmd. Caddy started before the agent was listening on :3100, returning 502 to all health checks. |
| `internal/templates/caddyfile.go` | Template changes can trigger ACME account creation. Let's Encrypt rate limits: 10 registrations per IP per 3 hours. |
| `cmd/specter/commands/image.go` | Provisioning script must call `sync` before snapshot. Without it, Bun (99MB) was captured as 0 bytes on disk. |

## Key Gotchas

### Deploy Sequence (order matters)

1. Create VM from golden snapshot (~1s)
2. Create DNS A record on Cloudflare (~1s, parallel with boot)
3. Wait for VM boot (68-180s depending on datacenter)
4. Wait for SSH availability (11-24s after VM reports running)
5. Deploy agent code via SSH + start systemd service
6. **Retry loop**: poll `localhost:3100/health` every 1s, up to 30 attempts
7. **Only after agent responds**: enable and start Caddy
8. Caddy provisions TLS via ACME HTTP-01 (~5-8s)
9. Health check `https://agent.domain.com/health` returns 200

Caddy MUST NOT start before the agent is listening on port 3100. This is the single most important invariant in the deploy flow.

### ReadWritePaths

The systemd unit uses `ProtectSystem=strict` and `ProtectHome=read-only`. The agent process can only write to paths listed in `ReadWritePaths`. Currently: `/home/specter/app`. If you add new write locations, they must be in this list or the agent will get permission denied at runtime.

### Sync Before Snapshot

Hetzner snapshots capture disk state at power-off. Unbuffered writes produce 0-byte files. The image build script must call `sync` before `cloud-init clean --logs`. This is how we lost Bun's 99MB binary in early testing.

### Caddy Disabled in Golden Image

Caddy is installed but stopped and disabled in the golden image. A port-80-only placeholder Caddyfile prevents ACME registrations on boot. The deploy script enables and starts Caddy after the agent is verified running. Do not change this order.

### DNS Must Not Be Proxied


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghostwright/specter](https://github.com/ghostwright/specter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
