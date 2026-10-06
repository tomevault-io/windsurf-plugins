---
trigger: always_on
description: bit is a self-hosted URL shortener: Crystal, Kemal, SQLite. `docs/openapi.yaml` is the API contract. `docs/SETUP.md` covers deployment and the benchmark.
---

# AGENTS.md

bit is a self-hosted URL shortener: Crystal, Kemal, SQLite. `docs/openapi.yaml` is the API contract. `docs/SETUP.md` covers deployment and the benchmark.

## Map

- `bit.cr`: entry point. `app/routes.cr`: routes and the JSON `error` handlers.
- `app/controllers/`: `link.cr` (API), `click.cr` (redirect with async click tracking via `spawn`), `ping.cr`.
- `app/middlewares/`: `auth.cr` (`X-Api-Key` on every path except `/api/ping` and `/:slug`), `cors.cr`.
- `app/lib/`: `database.cr` (Crecto repo, runs migrations at boot), `migrator.cr`, `errors.cr`, `ua_parser.cr` and `ip_lookup.cr` (bounded caches).
- `app/services/`: `slug.cr`, `cli.cr` (user management, admin bootstrap from `ADMIN_NAME` and `ADMIN_API_KEY`). `app/config/env.cr`: environment defaults.
- `db/migrations/*.sql`: micrate-format files. `app/lib/migrator.cr` applies them at boot and keeps state in `micrate_db_version`.
- `data/`: user-agent regexes and the GeoLite2 database, refreshed by `.github/workflows/update-parsers.yml`.
- `scripts/cli.cr` and `scripts/benchmark.cr`. `spec/` runs in-process; `e2e/` starts `./bin/bit` as a real process.

## Constraints that are easy to miss

- `ENV` at compile time decides whether dotenv loads. Build the production binary with `ENV=production`. Any other build loads `.env.<ENV>` at runtime (default `.env.development`), and that file must exist.
- Migrations: `bit.cr` and every `cli` command except `--migrate-down` apply pending `-- +micrate Up` sections first. `cli --migrate-down` runs the `-- +micrate Down` section of the newest applied version and refuses an empty one (`20250319192003` is irreversible). The current image re-applies a rolled-back file on its next start, so a rollback only sticks when an older image is deployed next. A statement ends at a line ending with `;`, and any other `-- +micrate` directive raises. Each file runs in `BEGIN IMMEDIATE` with foreign keys off, so a `DROP TABLE` on a parent table does not cascade, and it fails if it adds a foreign key violation. Orphan rows from before foreign keys were enforced are tolerated, except that `20250319192003` cannot convert orphan links or clicks. A failing migration stops the app at boot.
- `App::Lib::Database::URL` appends `journal_mode=WAL`, `synchronous=NORMAL`, `foreign_keys=true`, and `busy_timeout=100` to `DATABASE_URL`; the app, the migrator, and the CLI all connect with it, and crystal-sqlite3 runs them as PRAGMAs on every connection. A key already present in `DATABASE_URL` wins, because crystal-sqlite3 reads the first value. WAL mode is what makes the redirect path fast. Foreign keys enforce the `ON DELETE CASCADE` rules. SQLite's busy wait blocks the single-threaded scheduler, so the timeout stays short.
- Kemal 1.14 discards a response body written before an error. Raise an `App::*Exception` from `app/lib/errors.cr`; do not print a body and then raise.
- Every API error is JSON. Auth runs before routing, so an unmatched route returns 401 without a key and `404 {"error":"Resource not found"}` with one. A wrong method returns 405 with an `Allow` header.
- Updating a link's URL regenerates its slug; the old short URL stops working.
- CI (`.github/workflows/tests.yml`, pull requests only) runs `shards install`, the production build, `crystal spec`, and `crystal spec e2e`. Formatting is not enforced.

## Checks

Run from the repository root, cheapest first:

1. `shards install`
2. `ENV=production shards build --release --no-debug` (builds `bit`, `cli`, `benchmark` into `bin/`)
3. `ENV=test crystal spec`
4. `ENV=test crystal spec e2e` (needs the step 2 binary)
5. `docker build -t bit:local .` (add `--platform linux/amd64` to check the other published architecture)

## Manual verification

The specs cover most status codes in-process. The steps below check the published image over a real network, GeoIP, headers, load, and memory. Use them when a change touches a route, the Dockerfile, a dependency, or the redirect path.

The endpoint walkthrough repeats status codes the specs already assert, on purpose. The specs call the handler chain in memory through spec-kemal, with the development build, macOS SQLite, and no socket. The walkthrough sends the same requests to the production binary inside the Docker image, so it also catches what only exists there: the `alpine` runtime libraries, the GeoLite2 file and user-agent regexes packaged in the image, the compile-time `ENV=production` branch, the SQLite settings on a real file, and headers over HTTP. A status code that passes in the specs and fails in the walkthrough points at the image or the build, not the code.

### Run with Docker

```bash
docker build -t bit:local .
docker run -d --rm --name bit-local -p 4000:4000 \
  -e APP_URL=http://localhost:4000 -e ADMIN_NAME=Admin -e ADMIN_API_KEY=local-key bit:local
docker logs bit-local      # expect "New user created" and no errors
```

Stop it with `docker stop bit-local`. The database lives in the container and is removed with it, so restart it before repeating the walkthrough.

### Every endpoint, including edge cases


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sjdonado/bit](https://github.com/sjdonado/bit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
