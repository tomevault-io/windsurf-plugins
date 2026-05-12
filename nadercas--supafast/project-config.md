---
trigger: always_on
description: SupaFast is a self-hosted Supabase distribution deployed on Hetzner VPS via
---

# SupaFast — Claude project notes

SupaFast is a self-hosted Supabase distribution deployed on Hetzner VPS via
cloud-init. This file documents the parts a future Claude session is most
likely to get wrong if it reads the code cold.

## Repo layout (high level)

- `components/cloudInitGenerator.js` — browser-side generator that produces the
  one-shot Hetzner user-data script. Contains the VM bootstrap and the
  embedded host scripts `supabase-upgrade.sh` + `supabase-rotate-keys.sh`
  (installed by cloud-init because the management container needs them on
  first boot).
- `management/` — the management container (`ghcr.io/nadercas/supafast`). Runs
  inside the VM; drives rotation, upgrades, pin, and migrations via systemd
  path-unit triggers.
  - `management/server.js` — HTTP API.
  - `management/public/index.html` — single-page admin UI.
  - `management/migrations/NNN_*.sh` (+ sibling asset dirs) — numbered
    migrations bundled into the image.
  - `management/hostbin/` — frozen copies of host scripts shipped *inside* the
    image and installed to `/usr/local/bin/` by the bootstrap endpoint
    (`supafast-migrate.sh`, `supabase-pin-digests.sh`).
  - `management/scripts/extract-runner.js` — regenerates everything in
    `management/hostbin/` from the `get*Sh()` generators. Run this after
    editing any of those generators so the frozen copies stay in sync.

## Shipping a management-image change to production

Whenever `management/` changes (server.js, public/, migrations/, hostbin/,
Dockerfile) and needs to reach running servers:

1. If any `get*Sh()` generator in `cloudInitGenerator.js` was edited, run
   `node management/scripts/extract-runner.js` to refresh frozen copies in
   `management/hostbin/`.
2. Pick the next version tag (current latest on GHCR is `v1.3` — bump to
   `v1.4`, `v1.5`, …). Build and push **from inside `management/`** tagging
   both the version and `latest`:
   ```
   cd management
   docker buildx build --platform linux/amd64,linux/arm64 \
     -t ghcr.io/nadercas/supafast:vX.Y \
     -t ghcr.io/nadercas/supafast:latest \
     --push .
   ```
   Never skip the version tag — the admin panel's Upgrades dropdown lists
   `/v2/.../tags/list` and filters to `^v\d+(\.\d+){0,2}$`. Tags that don't
   match that regex don't appear.
3. Bump `IMAGE_MANAGEMENT=ghcr.io/nadercas/supafast:vX.Y` in
   `components/cloudInitGenerator.js` so fresh deploys pin to the new
   version. Commit generator + hostbin copies + any migration changes in
   one commit.
4. On each target server, pull and recreate mgmt:
   `cd /opt/supabase/docker && sudo docker compose pull management && sudo docker compose up -d management`
   If the server's `.env` still pins `IMAGE_MANAGEMENT=...@sha256:<digest>`
   from a prior panel upgrade, pulling by digest gets you the same image.
   Either use the panel's admin-panel Upgrade button (preferred) or
   `sudo sed -i 's|^IMAGE_MANAGEMENT=.*|IMAGE_MANAGEMENT=ghcr.io/nadercas/supafast:vX.Y|' .env` first.
5. If a host runner (`supafast-migrate.sh`, `supabase-pin-digests.sh`) changed,
   re-run the bootstrap endpoint so `/usr/local/bin/*.sh` and its systemd path
   unit get rewritten from the new image. Via docker bridge:
   `MGMT_IP=$(sudo docker inspect $(sudo docker compose ps -q management) -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' | head -1) && curl -fsSL http://$MGMT_IP:3001/api/migrations/bootstrap.sh | sudo bash`
6. If recreating mgmt, also `sudo docker compose restart caddy` — Caddy
   caches the container DNS and 502s until restarted.
7. Open the panel → Upgrades tab → Apply pending if a new migration shipped.

End-to-end sanity check after non-trivial changes: add a throwaway
`003_noop.sh` migration with just `echo noop`, bump `LATEST_MIGRATION_VERSION`,
build+push, pull on a server, confirm it shows as pending and applies.

**Known gap:** pulling a new image does not auto-re-run bootstrap. Runner
changes therefore require step 5 on existing servers. If this becomes painful,
add an entrypoint hook in the mgmt container that calls its own
`/api/migrations/bootstrap.sh` on startup — idempotent, safe.

## Cloud-init size budget

Hetzner caps `user_data` at 32 KiB. `minifyBashOutsideHeredocs` strips comments
from generated bash to stay under. When adding to the generator, check the
output size — keep 500+ bytes headroom.

## Migration system

SupaFast has two code-delivery channels. Knowing which one reaches which
servers is the thing to get right.

| Channel | What it ships | Reaches |
|---|---|---|
| `cloudInitGenerator.js` (GitHub) | VM bootstrap | **Fresh deploys only** — existing VMs never re-run cloud-init |
| `ghcr.io/nadercas/supafast` (management image) | Admin UI + migrations bundle + runner | **All servers** — pulled via the existing Upgrades tab |

So: new features that need to reach existing servers go through the management
image as a numbered migration. The generator is only touched when a fresh
deploy needs the feature built in from day one.

### Persistent artifacts on the VM

| Path | Purpose |
|---|---|
| `/opt/supabase/docker/.supafast-version` | Integer. Highest applied migration number. Fresh deploys seed this to `LATEST_MIGRATION_VERSION` so bundled migrations are not re-run. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nadercas/supafast](https://github.com/nadercas/supafast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
