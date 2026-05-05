---
trigger: always_on
description: **微信视频号下载助手** (WeChat Channel Video Downloader) is a Windows desktop application written in Go 1.23+ that:
---

# AI Coding Agent Instructions for wx_channel

## Project Overview
**微信视频号下载助手** (WeChat Channel Video Downloader) is a Windows desktop application written in Go 1.23+ that:
- Intercepts and downloads videos from WeChat Channels
- Automatically decrypts encrypted videos
- Provides batch download capabilities and a web console for management
- Uses HTTP proxy interception to hook into WeChat's JavaScript

## Architecture Overview

### Core Layering
```
CLI Entry (cmd/root.go) 
  ↓
App Orchestrator (internal/app/App)
  ↓
HTTP Proxy (SunnyNet library) 
  ↓
Request Interceptor Chain:
  - ScriptHandler: Modifies WeChat JS
  - UploadHandler: Manages downloads  
  - RecordHandler: Logs download history
  - BatchHandler: Bulk operations
  - APIRouter: REST API endpoints
  ↓
Service Layer (internal/services/): 
  - DownloadRecordService
  - StatisticsService
  - QueueService, etc.
  ↓
Data Layer (SQLite via GORM in internal/database/)
```

### Critical Components

| Component | Role | Key Files |
|-----------|------|-----------|
| **Proxy Interception** | Hooks HTTP requests using SunnyNet | `app.go`, `handlers/*.go` |
| **JavaScript Injection** | Injects custom JS to intercept WeChat video events | `internal/handlers/script.go`, `internal/assets/inject/` |
| **Download Engine** | Integrates Gopeed for download management | `services/gopeed_service.go` |
| **WebSocket Hub** | Real-time comms with web console | `internal/websocket/hub.go`, `handlers/websocket.go` |
| **Configuration** | Dynamic Viper-based config with DB override | `internal/config/config.go` |

## Essential Development Patterns

### 1. Handler Chain Pattern
Each request flows through multiple handlers that can intercept/modify it:
```go
// Handlers implement router.Interceptor interface
func (h *MyHandler) Handle(Conn *SunnyNet.HttpConn) bool {
    // Return true if handled, false to pass to next handler
}
```
Add new handlers to `internal/app/app.go` in the `requestInterceptors`/`responseInterceptors` chains.

### 2. Dependency Injection via App Constructor
All handlers receive dependencies through constructors:
```go
func NewUploadHandler(cfg *config.Config, wsHub *websocket.Hub, gopeedService *services.GopeedService) *UploadHandler
```
Services are singleton instances created in `internal/app/app.go` and shared across handlers.

### 3. JavaScript Interception Events
WeChat JS fires custom events via eventbus (defined in `internal/assets/inject/eventbus.js`):
- `PCFlowLoaded`: Video list loaded
- `FeedProfileLoaded`: Single video details loaded  
- `BeforeDownloadMedia`: Before video fetch
- `MediaDownloaded`: Video fetch complete

New handlers intercept these via regex replacements in JavaScript strings. See `script.go` for patterns.

### 4. Configuration Priority (Dynamic Reloading)
1. YAML config file (`config.yaml`)
2. Environment variables
3. Database settings table (allows runtime updates without restart)

When modifying config, use `config.Get()` to get current instance, not `Config` passed at startup.

### 5. WebSocket Communication Pattern
For web console updates:
```go
app.WSHub.Broadcast([]byte(`{"type":"download_progress","progress":45}`))
```
Clients receive real-time updates without polling.

## Critical Files to Understand

| File | Purpose | Read First? |
|------|---------|---|
| `internal/app/app.go` | Main App struct, initializes all components | ✅ Yes |
| `internal/handlers/script.go` | ~1500 lines, most complex - modifies WeChat JS | ⚠️ Complex |
| `internal/router/api_routes.go` | REST API routing and CORS | ✅ Yes |
| `internal/services/gopeed_service.go` | Download orchestration via Gopeed lib | When modifying downloads |
| `config.yaml.example` | Configuration schema | Reference |
| `web/` | Frontend source (Vue/HTML) | When fixing UI issues |

## Build & Testing

### Building
```powershell
# Standard build (Windows amd64)
go build -o wx_channel.exe

# With version info
go build -ldflags "-X wx_channel/internal/version.Version=v1.0.0" -o wx_channel.exe
```

### Key Dependencies
- `github.com/qtgolang/SunnyNet`: HTTP proxy interception (Windows only)
- `github.com/GopeedLab/gopeed`: Download engine integration
- `gorm.io/gorm`: ORM for SQLite database
- `github.com/spf13/cobra`: CLI framework
- `github.com/gorilla/websocket`: WebSocket support

### Testing
**No test files found.** When adding tests:
- Use Go's testing package in `*_test.go` files
- Mock SunnyNet/Gopeed dependencies
- Test service logic independently from HTTP handlers

## Common Tasks & Approaches

### Adding a New Configuration Option
1. Add field to `Config` struct in `internal/config/config.go`
2. Add YAML mapping via `mapstructure` tag
3. Set default in `loadConfig()` via Viper
4. Access via `config.Get()` in handlers

### Intercepting a New WeChat API
1. Find the JavaScript pattern in `script.go` 
2. Use regexp to locate the function (see `finderPcFlow` example)
3. Inject interrupt code that calls `WXU.emit(WXU.Events.YourEvent, data)`
4. Listen for event in Go handlers via request/response inspection

### Adding a New Download Endpoint
1. Create handler in `internal/handlers/`
2. Register in `APIRouter.registerRoutes()` 
3. Use `NewSunnyNetResponseWriter` to write responses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nobiyou/wx_channel](https://github.com/nobiyou/wx_channel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
