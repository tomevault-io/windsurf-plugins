---
trigger: always_on
description: Guidance for AI coding agents working on this repository.
---

# AGENTS.md

Guidance for AI coding agents working on this repository.

> **Security rules are not optional.** Read [`docs/security.md`](./docs/security.md) before writing or modifying any code that runs a subprocess, touches a credential, renders a template placeholder, or gates a destructive operation. `pnpm lint` enforces the mechanical subset via `scripts/check-shell-safety.js`.

## Project Overview

Vibecarbon is a CLI tool that generates production-ready software applications with Hono + Vite + React 19 + self-hosted Supabase. The repository contains:

1. **CLI tools** (`src/`): `cli.js` (entry point) + command modules (`create.js`, `add.js`, `remove.js`, `up.js`, `down.js`, `reset.js`, `push.js`, `deploy.js`, `destroy.js`, `status.js`, `backup.js`, `restore.js`, `failover.js`, `scale.js`, `configure.js`, `upgrade.js`, `activate.js`)
2. **Template directory** (`carbon/`): The complete template that gets copied and configured when users run `vibecarbon create`
3. **Test suite** (`tests/`): Comprehensive Vitest test suite

### CLI Commands
```bash
vibecarbon create <project-name>   # Create new project
vibecarbon add <feature>           # Add optional feature (observability, redis)
vibecarbon remove <feature>        # Remove a feature from project
vibecarbon up                      # Start local dev environment
vibecarbon down                    # Stop local dev environment
vibecarbon reset                   # Reset local environment (removes all data)
vibecarbon deploy [environment]    # Deploy an environment (-provider, -mode <compose|compose-ha|k8s|k8s-ha>, -region, -standby-region, -full, -restore, -allow-degraded)
vibecarbon destroy [environment]   # Tear down cloud environment
vibecarbon status                  # Show project and deployment status
vibecarbon backup [environment]    # Create or list database backups
vibecarbon restore [environment]   # Restore database from backup
vibecarbon failover [environment]  # Initiate failover to standby region (HA deployments)
vibecarbon scale [environment]     # Scale worker nodes and instance sizes
vibecarbon configure               # Configure external services and project settings (billing, OAuth, SMTP, CI/CD, globalization, etc.)
vibecarbon upgrade                 # Upgrade infrastructure files to latest template
vibecarbon activate <key>          # Activate a Fullerene license key
vibecarbon deactivate              # Remove the current license
vibecarbon shell [environment]     # Interactive bash with KUBECONFIG + cloud credentials exported
vibecarbon diagnose [environment]  # Dump full cluster state (nodes, pods, Flux, network) to ~/.vibecarbon/diag-*
vibecarbon console <node>          # Open the cloud provider's web console for a node
vibecarbon access [subcommand]     # Manage SSH + k8s-API operator-CIDR allowlist
```

### `destroy` exit codes

Teardown is best-effort per resource (one class failing never aborts the rest),
so the exit code — not the absence of red text — is the verdict. Every failed
delete, gated survivor, incomplete provider listing and thrown step is tallied
in a leak ledger (`src/lib/destroy/leak-ledger.js`) and printed as a leak report
at the end: one line per surviving resource (`LEAK` / `UNVERIFIED` / `FOREIGN` /
`AT-RISK`, class, identity, why it survived), then a summary count. A clean
destroy prints a one-line all-clear.

One survival is sanctioned and deliberately OUTSIDE the ledger: the dedicated
Pulumi state bucket is KEPT on destroy (announced in its own step line, deleted
only with `-purge`). Recreating a just-deleted same-name bucket is how acked
state writes were lost on 2026-08-07, and a warm bucket is what a redeploy
resumes against; a kept-by-design bucket is neither a leak, unverified,
foreign, nor a predicted leak, so it takes none of those labels.

| Code | Meaning | Scripted caller should |
| --- | --- | --- |
| `0` | Clean: everything confirmed deleted (or already absent), every listing read in full. `FOREIGN` (proven not ours) and `AT-RISK` (config predicts a leak) lines are reported here without failing. | proceed |
| `1` | The destroy could not run: bad flags, no API token, no such environment, cancelled, unhandled throw. **Nothing was torn down.** | treat as a hard failure |
| `2` | The teardown ran to completion but **leaked**, or could not verify a class. Resources may still be billing. | report what leaked (the report lines are greppable — `isLeakReportLine`) |

`FOREIGN` is exit-neutral on purpose: a `pvc-*` volume absent from a complete
capture of this environment's own PersistentVolumes belongs to someone else
(a parallel rig), and failing our build for a disk we are forbidden to delete
would make the signal unreadable.

## Testing

4 tiers: `unit`, `integration`, `loadtest`, `e2e`. See `docs/tests.md` — including its **guard decision procedure** (given a change, which guard class must accompany it) and the **parity rule**: a bugfix on one provider/tier/path must prove every sibling surface is fixed or genuinely unaffected.

```bash
pnpm test                    # All non-e2e tiers
pnpm test:unit               # Unit (pure, in-process; ~10s)
pnpm test:integration        # Integration (spawns CLI, fixtures, cloud stubbed; ~1min)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperformant/vibecarbon](https://github.com/hyperformant/vibecarbon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
