---
trigger: always_on
description: ZCP control-plane container `zcp` inside this Zerops project. `zerops_*` MCP = primary surface for state/lifecycle/deploy/env/logs/verify. Bash/npx/SSH/zcli/psql/mysql/redis-cli = escape hatches for things `zerops_*` doesn't cover.
---

<!-- ZCP:BEGIN -->
# Zerops

ZCP control-plane container `zcp` inside this Zerops project. `zerops_*` MCP = primary surface for state/lifecycle/deploy/env/logs/verify. Bash/npx/SSH/zcli/psql/mysql/redis-cli = escape hatches for things `zerops_*` doesn't cover.

**Env here:** the project's vars — including each managed service's connection vars (`$db_connectionString`, `$cache_connectionString`, …) — are in this container's shell once the service is provisioned. Run ad-hoc ops in place by name: `psql "$db_connectionString" -c '…'` (mask when inspecting: pipe through `sed 's/=.*/=<set>/'`). Inside a runtime over `ssh`, reference the name THAT runtime wired (e.g. `$DATABASE_URL`, live after its first deploy) in a single-quoted body — not a sibling's bare `${db_*}`, which the default service isolation doesn't inject.

After bootstrap or adopt provision closes, service code SSHFS-mounts at `/var/www/{hostname}/` — the mount IS the service's runtime filesystem. **Edit** files there with Read/Edit/Write, not SSH. **Run** build/test/framework commands (`npm run build`, `npm test`, `php artisan`, `pytest`, DB migrations) INSIDE the service over SSH — `ssh {hostname} "cd /var/www && <cmd>"` — because the runtime and its dependencies live in the service container, not on this host; there the same code sits at `/var/www` (no `{hostname}` segment). That SSH is the expected path for running code, not an escape hatch. Long-running dev servers are the one exception — start them via `zerops_dev_server`, never a backgrounded `ssh "… &"` (the channel dies with the call). Missing a CLI/tool? `zerops` has passwordless sudo on the container — install it ad-hoc (`ssh {hostname} "sudo apk add <pkg>"` on Alpine, `sudo apt-get install` on Debian — `cat /etc/os-release` if unsure) rather than working around its absence. That install is **ephemeral** (gone on the next deploy = fresh container); a tool the build or runtime needs durably goes in `prepareCommands` in `zerops.yaml`. That ad-hoc affordance covers the app's OWN tooling only — anything under `/opt/zerops/**` is platform-owned; a broken or missing binary there (e.g. `zcli`) is report-not-repair: never patch it with a compat shim, reinstall, or replacement.

`zerops.yaml` lives at `/var/www/{hostname}/zerops.yaml`; per-service rules MAY exist at `/var/www/{hostname}/CLAUDE.md` — read if present. If `ls /var/www/{hostname}/` is empty, the service hasn't been bootstrapped yet — run `zerops_workflow action="start" workflow="bootstrap" route="adopt"` first.

## Zerops onboarding

When the user asks to be onboarded to Zerops — the exact phrase "onboard me to Zerops"
(any capitalization/punctuation), or a clear meta-onboarding request ("get me started
with Zerops", "I'm new here — what now?") — run the onboarding conversation before the
routing below. A request to get started with a SPECIFIC technology or task ("help me
get started with PostgreSQL", "deploy this repo") is normal routing, not onboarding.

1. Fetch `zerops_knowledge uri="zerops://playbooks/onboarding"` once and follow it.
2. Greet and offer its fork immediately — the opening needs no other tool call. Read-only
   state checks come after the person answers (or when they ask what's here); don't
   provision, import, or mutate anything until they pick a direction.
3. Once the user chooses to build or bring an app, normal routing (and the guided skill,
   when present) owns the work — onboarding only opens the conversation.
4. If Zerops tools are unavailable or auth fails, say so plainly and surface the reported
   recovery — never simulate onboarding.

Zerops has its own syntax. Don't guess — look up via `zerops_knowledge`, inspect live state via `zerops_*`. Runtime code runs in Zerops containers, not here.

## Route every user turn

| Intent | First action | Don't |
|---|---|---|
| Build/edit/scaffold/fix/deploy/debug a service | `zerops_discover`/`zerops_workflow action="status"` first if target/session unclear, then `zerops_workflow action="start" workflow="develop" intent="..." scope=["<host>"]` | Write code, run Bash/npx/SSH, or scaffold to scratch dirs before workflow start |
| No service yet, or infra/topology change — INCLUDING "deploy / set up / scaffold from existing recipe X" (user names a recipe slug like `zerops-laravel-minimal`) | `zerops_workflow action="start" workflow="bootstrap" intent="..."` — the route-menu surfaces the matching recipe; pick `route="recipe"` with the named slug | Write app code in bootstrap |
| Read or set platform state — logs/env/status/scale/subdomain/manage/events/verify | matching `zerops_*` tool | Guess values when live state exists |
| Promote dev/stage to a separate prod project ("go live", "deploy to prod", "nasaď na prod") | `zerops_workflow action="start" workflow="launch-production" intent="..." targetService="<host>"` | `zcli project create` or hand-rolled import.yaml |
| Pure concept Q unrelated to this project | prose, no tool | Re-route when user pivots to build/change |

## Discovery floor


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bhuvaneshmp15-glitch/ayursutra](https://github.com/bhuvaneshmp15-glitch/ayursutra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
