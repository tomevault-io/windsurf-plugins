---
trigger: always_on
description: Go native cross-platform desktop automation: mouse, keyboard, screen, bitmap, process, window handle, clipboard, and global event listener. Supports macOS, Windows, Linux; amd64 and arm64.
---

# RobotGo

Go native cross-platform desktop automation: mouse, keyboard, screen, bitmap, process, window handle, clipboard, and global event listener. Supports macOS, Windows, Linux; amd64 and arm64.

Module: `github.com/go-vgo/robotgo` — `go.mod` declares `go 1.25.0` (CI sets up Go 1.26.x).

## Build/Test/Lint Commands

Prerequisites: `GCC` must be installed. `CGO_ENABLED=1` (default). On macOS, Xcode Command Line Tools + Accessibility/Screen Recording permissions. On Linux, X11 + XTest (`libx11-dev xorg-dev libxtst-dev`).

- **Build**: `go build -v .`
- **Build all subpackages**: `go build -v ./...`
- **Fetch deps**: `go get -v -t -d ./...`
- **Test (CI minimal — no display required)**: `go test -v robot_info_test.go`
- **Test (full)**: `go test -v ./...` (Linux CI wraps with `xvfb-run` — see `.circleci/config.yml`)
- **Single test**: `go test -v -run TestGetScreenSize .`
- **Format**: `gofmt -w .` (code uses tab indentation, standard `gofmt` style)
- **Vet**: `go vet ./...`
- **Run an example**: `cd examples/mouse && go run main.go`

There is no Makefile / Taskfile / linter config. CI is `.github/workflows/go.yml` (macOS + Windows, Go 1.26.x: `go build -v .` then `go test -v robot_info_test.go` only) and `.circleci/config.yml` (Linux full tests under xvfb). The old `appveyor.yml` has been removed.

## Architecture

Single Go package `robotgo` at repo root (flat layout) with platform-specific files and C-binding subpackages. The default backend is Cgo wrappers over C headers vendored in subdirectories; build tags split platform implementations. In addition, three **pure-Go (no-Cgo) backends** now live in their own packages and are selected via build tags on the root package: `win/` (`-tags win`, Win32 via tailscale/win), `wayland/` (`-tags wayland`, wlroots virtual-input protocols), and `libei/` (`-tags libei`, xdg-desktop-portal RemoteDesktop for GNOME/KDE). `robotgo.go` carries `//go:build !wayland && !win && !libei` so exactly one backend compiles.

```
robotgo/
├── robotgo.go              # default Cgo API + preamble; //go:build !wayland && !win && !libei
├── robotgo_pub.go          # portable pkg vars: Version, MouseSleep, KeySleep, DisplayID, Scale...
├── doc.go                  # package doc
├── robotgo_mac.go          # //go:build darwin
├── robotgo_mac_unix.go     # //go:build darwin || linux
├── robotgo_mac_win.go      # //go:build darwin || windows
├── robotgo_win.go          # //go:build windows
├── robotgo_x11.go          # //go:build linux
├── robotgo_android.go, robotgo_adb.go
├── robotgo_ocr.go          # //go:build ocr (gosseract OCR)
├── libei.go                # //go:build linux && libei — wires libei/ backend into robotgo pkg
├── wayland_n.go, windows_n.go
├── key.go, keycode.go, screen.go, img.go, ps.go
├── robotgo_fn_v1.go        # deprecated v1 aliases (kept for compat)
├── robot_info_test.go      # only portable test (used by GitHub Actions)
├── robotgo_test.go         # full interactive tests
├── base/       # C helpers (MMBitmap, rgb, microsleep, types, os, pubs, xdisplay)
├── mouse/      # Go pkg + C (mouse.h, mouse_c.h) with *_darwin.go/_windows.go/_x11.go
├── key/        # Go pkg + C (keycode.h, keycode_c.h, keypress.h, keypress_c.h, key_windows.go)
├── screen/     # Go pkg + C (goScreen.h, screen.go, screen_c.h, screengrab_c.h)
├── window/     # Go pkg + C (goWindow.h, window.h, alert_c.h, win_sys.h, pub.h)
├── clipboard/  # Go pkg (darwin/unix/windows variants) + cmd/gocopy, cmd/gopaste, example/
├── win/        # pure-Go Windows backend (no Cgo); //go:build windows
├── wayland/    # pure-Go wlroots Wayland backend; internal/protocols/ (wlr_*), libei/
├── libei/      # pure-Go libei/xdg-portal backend (GNOME/KDE); //go:build linux
├── mcp/        # MCP server package (mcp.go, currently a stub)
├── event/      # C headers for android/ios global hooks (event_c.h)
├── cv/         # OpenCV helper (gocv.go)
├── examples/   # main.go + mouse, key, screen, window, scale — runnable main.go
├── lang/       # translated READMEs (de, es, fr, ja, ko, pt, ru, zh, zht)
├── skills/     # SKILL.md (agent skill descriptor)
├── x11/, darwin/   # currently empty placeholder dirs
├── docs/       # install.md, keys.md, CHANGELOG.md, README.md, archive/
└── .github/workflows/go.yml, .circleci/config.yml
```

Key subpackage relationships: the root `robotgo` package pulls C code from `screen/goScreen.h`, `mouse/mouse_c.h`, `window/goWindow.h`. The `key/` and `clipboard/` directories are importable Go packages; `base/` is header-only C support. The pure-Go `win/`, `wayland/` and `libei/` packages each mirror the robotgo API surface (mouse/keyboard/screen/window/process) so a backend can be swapped per platform with a build tag.

## Code Style

- **Copyright header**: every Go and C file starts with the 10-line `Copyright (c) 2016-2026 AtomAI...` block (see `CONTRIBUTING.md`). Preserve it verbatim when editing; add a second header only if authorship changes.
- **Indentation**: tabs (Go default). Run `gofmt` before committing.
- **Build tags**: use both forms together — `//go:build darwin` plus legacy `// +build darwin` — matching existing files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-vgo/robotgo](https://github.com/go-vgo/robotgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
