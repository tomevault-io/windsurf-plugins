---
trigger: always_on
description: Operator/agent contract for staging and prod. Full procedures live in [`README.md`](README.md); do not duplicate them here.
---

# AGENTS.md — deploy (DigitalOcean + Compose)

Operator/agent contract for staging and prod. Full procedures live in [`README.md`](README.md); do not duplicate them here.

## Topology (4 droplets, NYC1)

| Host | Role | Gateway |
|------|------|---------|
| `base-staging` | staging master | yes (`role-master` + `env-staging`) |
| `base-staging-validator` | staging validator | no — VPC → staging master `:8080` |
| `base-prod` | prod master | yes (`role-master` + `env-prod`) |
| `base-prod-validator` | prod validator | no — VPC → prod master `:8080` |

Terraform: [`terraform/`](terraform/). Firewall: SSH from operator IP; CI uses ephemeral `/32` via `.github/actions/do-firewall` (always tear down). Spaces for Postgres backups (promote/restore).

## Compose matrix

`remote-deploy.sh --env staging|prod --role master|validator` stacks:

| File | Purpose |
|------|---------|
| `compose/role-master.yml` | gateway profile, VPC publish; **no validator** (avoids dual CRV4 submit) |
| `compose/role-validator.yml` | no gateway; external gateway endpoint; sole on-chain submitter |
| `compose/env-staging.yml` | testnet 541, faster coordination |
| `compose/env-prod.yml` | mainnet, conservative intervals |
| `compose/env-local.yml` | **local only** — ports/smoke knobs/tunnel env; always on top of `env-staging` |

Verify: `./deploy/scripts/assert-compose-matrix.sh`.  
Root `docker-compose.staging-*.yml` overrides are **obsolete** — use `deploy/compose/` only.  
`remote-deploy.sh` never selects `env-local*.yml`.

## Postgres vs ephemeral state

Compose always runs a digest-pinned `postgres` service (`base-pgdata` volume, healthcheck, `deploy/env/postgres.env`). App `BASE_DATABASE_URL` must match that file (materialize via `./deploy/scripts/materialize-env.sh`; local-e2e also injects `LOCAL_DATABASE_URL` from it).

| Data | Store |
|------|--------|
| Design harnesses / runs / stages / artifacts metadata / admin rounds | **Postgres** (`design_*`) |
| Prism submissions / stage events | **Postgres** (`prism_*`) |
| Gateway raw weight leaves + sealed bundles | **Postgres** (`raw_weight_snapshot`, `epoch_bundle`, …) |
| Validator attestations (when DB configured) | **Postgres** |
| Design sandbox staging files | volume `${BASE_STATE_DIR}/design/staging` + `design-artifacts` |
| Gateway challenge **backend registry** | **in-memory** — re-seed after gateway restart (`remote-deploy.sh` does this on master) |
| site-api (`GET /v1/site/*`) | no DB — proxies challenge upstreams via gateway |
| Unit/integration tests | may construct `Memory*Store` directly; omit `BASE_DATABASE_URL` only there |

Migrations (`crates/db/migrations`) run on boot in gateway / design-challenge / prism-challenge when `BASE_DATABASE_URL` is set. Compose requires `deploy/env/{design,prism}-challenge.env` so challenges cannot silently boot on memory.

Verify rows (local master stack):

```bash
docker compose -f docker-compose.yml exec -T postgres \
  sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -c \
  "SELECT COUNT(*) FROM design_harness; SELECT COUNT(*) FROM prism_submission;"'
```

## Local testnet E2E

Full procedure: [`docs/runbooks/local-testnet-e2e.md`](../docs/runbooks/local-testnet-e2e.md).

```bash
./deploy/scripts/materialize-env.sh
./deploy/scripts/local-e2e.sh --dry-run          # plan + compose render
./deploy/scripts/local-e2e.sh --smoke            # healthz + weights seal smoke + tunnel
./deploy/scripts/local-e2e.sh --live             # owner wallet + REQUIRE_OWNER=1
./deploy/scripts/local-e2e.sh --down
```

| Prereq | smoke | live |
|--------|-------|------|
| Docker, Compose v2 | yes | yes |
| `cloudflared` (or `--no-tunnel`) | yes | yes |
| `deploy/env/*.env` (examples OK) | yes | yes |
| `gateway_sk` (seal) + `prism_sk` / `design_sk` (leaf sigs; pubs ↔ trust root) | yes (prefer `~/.base-secrets/challenge-*.sk`) | real preferred |
| `deploy/secrets/wallets/base-owner` | **no** (not needed for `/v1/weights/latest`) | **yes** (netuid 541 owner) |
| `base-validator` wallet | **no** (fetch-only) | for on-chain weight submit |
| Fresh `target/release/{gateway,validator,…}` (or `BASE_DOCKER_BUILD_FROM=source`) | recommended | **required** for real chain |

**Weights seal smoke (default on `--smoke`):** after healthz, `local-e2e.sh` runs `weights-smoke` — signed prism leaves for the live metagraph → `POST /v1/admin/seal` → assert `GET /v1/weights/latest` is **200** with **`sealed: true`**. Skip with `--no-weights-smoke`. Pre-seal, latest is **200 burn** (`sealed: false`, uid 0 = 100%) — never 404; that is unrelated to a missing gateway owner wallet. Prefer `--burn` on mainnet when sealing without real challenge scores (all `NoScore` → uid 0).

**Interim prod burn seal (until prism auto-emits):** keep a fresh sealed bundle on the master gateway so validators can Match + CRV4 submit. On the prod master this runs as a **systemd timer** (`base-burn-seal.timer`, every 21 min — above the 100-block `WeightsSetRateLimit`, inside the ~256-block Finney state-pruning window) driving [`scripts/prod-burn-seal.sh`](scripts/prod-burn-seal.sh); units live in [`systemd/`](systemd/). Install:

```bash
install -m 0755 target/release/weights-smoke /opt/base/bin/weights-smoke

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlatformNetwork/platform](https://github.com/PlatformNetwork/platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
