---
trigger: always_on
description: Guidance for AI coding agents working in the `iptv-tool-v2` repository.
---

# AGENTS.md

Guidance for AI coding agents working in the `iptv-tool-v2` repository.

## Project Overview

IPTV management tool with a Go 1.25+ backend (Gin + GORM + SQLite) and an embedded Vue 3 frontend (Element Plus + Vite). The server aggregates live TV channel sources, fetches EPG data, and publishes subscriptions in M3U/TXT/XMLTV/DIYP formats. The Vue SPA is embedded into the Go binary at build time via `//go:embed` in `web/embed.go`. Full i18n support (zh, en, zh-Hant) with backend-driven locale loading.

## Repository Structure

```
cmd/iptv-server/main.go    # Entrypoint (CLI flags, init order, server start)
internal/
  api/                      # Gin HTTP handlers, router setup, middleware, rate limiting
    access_control.go        # IP whitelist/blacklist CRUD controller
    acl_middleware.go        # Access control middleware with cached evaluation
    epg_source.go            # EPG source CRUD controller
    geoip.go                 # GeoIP settings controller (status, download, auto-update)
    live_source.go           # Live source CRUD + sync/detect trigger controller
    log.go                   # Runtime & access log ring buffers + API handlers
    log_middleware.go         # Access logging middleware (records to ring buffer + AccessStat)
    logo.go                  # Channel logo upload/CRUD controller
    publish.go               # Publish interface CRUD + preview controller
    ratelimit.go             # Sliding-window IP rate limiter on login
    router.go                # Central route registration (SetupRouter)
    rule.go                  # Aggregation rule CRUD controller
    settings.go              # Detection settings + ffprobe upload controller
    system.go                # Init, login, password change, key crack controller
    update.go                # GitHub release check-update handler
  iptv/                      # IPTV platform client abstractions
    http.go                   # Shared HTTP client with rate-limiting semaphore
    interface.go              # IPTVClient interface definition
    types.go                  # Shared types (Channel, EPGProgram, etc.)
    huawei/                   # Huawei IPTV platform implementation + EPG strategies
    zte/                      # ZTE placeholder (empty)
  model/                     # GORM models (models.go) and DB init (db.go); global model.DB
  publish/                   # Aggregation engine, /sub/ handlers, and in-memory cache
    cache.go                  # Generic TTL cache with per-key mutex (stampede prevention)
    engine.go                 # Channel/EPG aggregation with rule application
    handler.go                # /sub/live/:path and /sub/epg/:path handlers
  service/                   # Business logic
    access_stat.go            # IP access statistics with async batched DB writes
    detect.go                 # Channel availability detection (ffprobe worker pool)
    epg_source.go             # EPG source sync service
    geoip.go                  # GeoIP database download/extract/lookup (GeoLite2 mmdb)
    iptv_lock.go              # Per-LiveSource mutex registry (prevents concurrent IPTV sessions)
    live_source.go            # Live source sync service
    url_selector.go           # Multicast/unicast URL priority selection for detection
    user.go                   # User registration, login, password, credential reset
  task/                      # Interval-based task scheduler (native Go, no cron library)
  version/                   # Build-time version injection + semver comparison
pkg/
  auth/                      # JWT generation/parsing (jwt.go), RSA key management (rsa.go)
  epg/                       # XMLTV parsing and generation
  i18n/                      # Internationalization engine
    i18n.go                   # Locale loading, Accept-Language negotiation (golang.org/x/text/language)
    middleware.go             # Gin middleware: resolves lang from X-Language / Accept-Language header
  logger/                    # slog setup with lumberjack + web UI log tee (LogAppender interface)
  m3u/                       # M3U and TXT/DIYP parsing and generation
  utils/                     # Utilities
    crack.go                  # Brute-force IPTV password cracker
    crypto.go                 # 3DES-ECB crypto for IPTV auth
    password.go               # Cryptographically random password generation
    sort.go                   # Natural sort comparison (e.g. "CCTV-2" < "CCTV-10")
locales/                     # Backend i18n locale files (embedded via //go:embed)
  embed.go                    # Embeds *.json locale files as embed.FS
  zh.json                     # Simplified Chinese (frontend + backend keys)
  en.json                     # English
  zh-Hant.json                # Traditional Chinese
web/                         # Vue 3 frontend (JS, not TypeScript)
  src/
    api/index.js              # Axios instance with auth interceptors
    composables/usePolling.js # Vue composable for periodic polling
    i18n/index.js             # vue-i18n setup with backend-driven locale loading
    layout/Layout.vue         # Main app layout (sidebar + header + content)
    router/index.js           # Hash-based routing with auth guards

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [super321/iptv-tool](https://github.com/super321/iptv-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
