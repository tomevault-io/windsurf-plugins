---
trigger: always_on
description: keep shipping; keep CI green
---

hi soup
keep shipping; keep CI green

# AGENTS

Owner: soup (s0up4200@pm.me)

## Progress Log

### 2026-02-21
- CI lint stabilization for oversized PR diff (`#82`)
  - synced `lint.yml` with qui baseline, then validated failure mode on this PR
  - issue confirmed: `golangci-lint-action` `only-new-issues: true` requests PR patch; GitHub returns `406` when diff > `20000` lines; action falls back to full-repo lint
  - applied oversized-PR fallback in backend lint step:
    - checkout pinned to `pull_request.head.sha`
    - `only-new-issues: false`
    - `args: --new-from-rev=HEAD~1`
  - result: backend lint remains incremental per pushed commit and avoids diff-API hard limit
  - CI status after fallback:
    - `Lint #22263181003`: success
    - `build #22263181007`: success

### 2026-02-20
- Dependency + security sweep (items #1 + #2)
  - upgraded backend direct dep:
    - `modernc.org/sqlite` `v1.46.0 -> v1.46.1`
  - upgraded frontend deps:
    - `tailwindcss` `4.1.18 -> 4.2.0`
    - `@tailwindcss/postcss` `4.1.18 -> 4.2.0`
    - `@types/node` `25.2.3 -> 25.3.0`
  - dependency classification fix:
    - moved `vite-plugin-pwa` from `dependencies` to `devDependencies` (build-time only)
  - security results:
    - `govulncheck ./...`: no vulnerabilities found
    - `pnpm -C web audit --prod`: no known vulnerabilities found
    - remaining `pnpm -C web audit` findings are dev-only and trace to ESLint 9 transitive chain (`minimatch@3`, `ajv@6`)
  - triage:
    - open GH dependabot alerts include stale/default-branch lockfile issues (e.g. axios) no longer present on this branch
    - next security step: dedicated ESLint 10 migration slice to clear remaining dev-only advisories
- ARR DRY pass (backend + frontend)
  - frontend:
    - added shared `ArrQueueStats` wrapper (`web/src/components/services/common/ArrQueueStats.tsx`)
    - removed four near-identical service wrappers:
      - `web/src/components/services/sonarr/SonarrStats.tsx`
      - `web/src/components/services/radarr/RadarrStats.tsx`
      - `web/src/components/services/lidarr/LidarrStats.tsx`
      - `web/src/components/services/readarr/ReadarrStats.tsx`
    - `ServiceCard` now routes sonarr/radarr/lidarr/readarr through the shared component
  - backend:
    - added shared queue hash/log helper `compareAndLogArrQueueChanges` (`internal/api/handlers/arr_queue_hash.go`)
    - sonarr/radarr/lidarr/readarr handlers now use shared helper; removed duplicated per-handler queue hash logging methods
    - removed duplicated per-handler queue broadcast one-liners and inlined shared publish path
  - verification:
    - `pnpm -C web lint`
    - `pnpm -C web typecheck`
    - `pnpm -C web test`
    - `pnpm -C web build`
    - `go test ./...`
- Web API client legacy timeout cleanup
  - removed dead per-service timeout map in `web/src/utils/api.ts` (old polling-era endpoints no longer used)
  - simplified timeout policy to:
    - default: `8000ms`
    - config health validation (`/api/health/:instance`): `12000ms`
  - switched timeout override selection to nullish (`customTimeout ?? ...`)
  - ensured abort timer cleanup runs on all fetch outcomes (`try/finally`)
- Docs item #3 completed (docs hardening)
  - added `docs/services_matrix.md` with current support matrix:
    - CLI group, discovery key, credential type/required-ness
    - detail endpoints and poll intervals from poller jobs
  - added `docs/k8s_discovery_example.yaml`:
    - ServiceAccount + ClusterRole + ClusterRoleBinding
    - annotated Service examples (`radarr`, `traefik`, `general`)
    - env placeholder pattern for `${VAR}` annotation substitution
  - linked new docs from:
    - `docs/config_management.md`
    - `README.md` service discovery section
- Docs parity sweep (supported services + k8s + CLI syntax)
  - `docs/commands.md`
    - corrected command-group name to `generic` (not `general`)
    - corrected generic add/remove/list examples
    - corrected tailscale add signature (`dashbrr service tailscale add <api-key>`)
    - corrected Maintainerr example port (`6246`)
    - added parameter notes for `generic` and tailscale URL behavior
  - `docs/config_management.md`
    - added k8s RBAC minimum for in-cluster service discovery
    - added missing `DASHBRR_GENERAL_API_KEY` env var
    - added full supported discovery service-type list
  - `README.md`
    - refreshed supported service inventory (media/download/network/infra groups)
- Plex/Jellyfin UI DRY alignment
  - new shared playback helpers: `web/src/components/services/common/playbackUi.tsx`
    - duration formatting (ms/ticks), bitrate formatting (kbps/bps), media/device icons, progress percent
  - `PlexStats` moved to shared helpers (dedupe only; no behavior change intent)
  - `JellyfinStats` refactor toward Plex-like active stream presentation
    - consistent stream tile layout, progress, and badges
    - fixed hook ordering to avoid conditional-hook regression
- Verification
  - `pnpm -C web lint`
  - `pnpm -C web typecheck`
  - `pnpm -C web test`
  - `pnpm -C web test:browser`
  - `pnpm -C web build`
  - `go test ./...`
- Jellyfin API parity pass (repo context: `~/github/oss/jellyfin`)
  - verified upstream session payload supports richer playback fields:
    - `NowPlayingItem.MediaStreams`
    - `PlayState.AudioStreamIndex`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autobrr/dashbrr](https://github.com/autobrr/dashbrr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
