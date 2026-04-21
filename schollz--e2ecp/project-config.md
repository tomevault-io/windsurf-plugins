---
trigger: always_on
description: **e2ecp** is a secure, end-to-end encrypted file transfer application that enables peer-to-peer file sharing through a web interface or command-line interface. The application uses ECDH P-256 key exchange and AES-GCM authenticated encryption, with a knowledge-free websocket-based relay server that cannot access transferred data.
---

# e2ecp - Copilot Agent Instructions

## Repository Overview

**e2ecp** is a secure, end-to-end encrypted file transfer application that enables peer-to-peer file sharing through a web interface or command-line interface. The application uses ECDH P-256 key exchange and AES-GCM authenticated encryption, with a knowledge-free websocket-based relay server that cannot access transferred data.

**Repository Stats:**
- **Type:** Go application with React web frontend
- **Size:** ~9 main Go source files (plus tests), 4 React components
- **Languages:** Go 1.25, JavaScript/React 19.2.0, Node.js 20+
- **Frameworks:** Cobra CLI, Vite, Gorilla WebSocket, React
- **Runtime:** Go 1.25+ required
- **Build time:** ~2-3 seconds for full build

## Recent Major Features (2024-2025)

The following significant features have been added recently:

1. **SHA256 Hash Verification** (#40) - File integrity checking via SHA256 hashes
2. **Zero-Knowledge Metadata Encryption** (#38) - File metadata (name, size, hash) encrypted before relay transmission
3. **Folder Support** - Automatic zip/unzip for folder transfers, supports multiple files
4. **QR Code Generation** - Terminal and web-based QR codes for easy room sharing (rsc.io/qr)
5. **Drag and Drop** - Improved web UI with drag-and-drop file/folder support
6. **Overwrite Protection** - --force flag prevents accidental file overwrites
7. **Per-IP Room Limits** - --max-rooms-per-ip flag prevents abuse
8. **Path Traversal Protection** (#10) - Security fix for file reception
9. **Protobuf Protocol** (#8) - 5-75x performance improvement over JSON

**New Dependencies:**
- Go: `rsc.io/qr` (QR code generation)
- Web: `jszip` (folder handling), `qrcode.react` (QR display), React 19.2.0

## Build & Development Workflow

### Prerequisites
- **Go:** 1.25 or higher (specified in go.mod)
- **Node.js:** 20+ (CI uses Node 24)
- **npm:** 10+ (bundled with Node.js)

### Initial Setup (CRITICAL - Always Run First)

**ALWAYS install web dependencies before any build operation:**

```bash
cd web && npm install
```

**Why this is critical:** The Makefile's `make server` target runs `make web` which calls `npm run build`. If `node_modules` is missing, you'll get `sh: 1: vite: not found` error and build will fail with exit code 127.

### Build Commands (In Order)

1. **Install web dependencies (required first):**
   ```bash
   cd web && npm install
   ```
   - Takes ~5 seconds
   - Creates `web/node_modules/`
   - Must be run before building

2. **Build web assets:**
   ```bash
   make web
   # OR
   cd web && npm run build
   ```
   - Takes ~1.5 seconds
   - Produces: `web/dist/` directory with bundled assets
   - Outputs: index.html, CSS, JS bundles, and font assets
   - Note: Vite warns about eval in protobufjs - this is expected and safe

3. **Build Go server (includes web build):**
   ```bash
   make server
   # OR
   make build  # Alias for server
   ```
   - Takes ~2-3 seconds total
   - Automatically runs `make web` first
   - Produces: `e2ecp` binary in root directory
   - Binary embeds web assets via `//go:embed all:web/dist`

4. **Clean build artifacts:**
   ```bash
   make clean
   ```
   - Removes `e2ecp` binary and `web/dist/` directory
   - Does NOT remove `web/node_modules/`

### Testing

**Run all tests:**
```bash
go test -v -cover ./...
```
- Takes ~3-4 seconds
- Tests are in: `main_test.go`, `integration_test.go`, `src/*/client_test.go`, `src/*/crypto_test.go`, `src/*/relay_test.go`
- Integration tests start a relay server and test full file transfer
- All tests should pass on clean checkout
- Coverage: ~42-48% overall

**Run without integration tests:**
```bash
go test -v -short ./...
```

**Run specific package tests:**
```bash
go test -v ./src/relay
go test -v ./src/client
go test -v ./src/crypto
```

### Development Server

**Web development server (for UI changes):**
```bash
cd web && npm run dev
```
- Starts Vite dev server on port 5173
- Hot-reload enabled
- Does NOT start the Go relay server

**Go development with hot-reload:**
```bash
air
```
- Uses `.air.toml` configuration
- Automatically rebuilds on Go file changes
- Runs `make server` on changes
- Starts server with `./e2ecp serve`

### Running the Application

**Start relay server:**
```bash
./e2ecp serve --port 8080
# Optional flags:
# --max-rooms <n>         # Maximum number of rooms (default: unlimited)
# --max-rooms-per-ip <n>  # Maximum rooms per IP address (prevents abuse)
```

**Send a file or folder:**
```bash
./e2ecp send <file-or-folder> [room-name]
# Folders are automatically zipped for transfer
# Displays QR code in terminal for easy room sharing
```

**Receive a file or folder:**
```bash
./e2ecp receive [room-name]
# Folders are automatically extracted
# Optional flags:
# --output <path>   # Specify output location
# --force          # Overwrite existing files without prompting
```

**View help:**
```bash
./e2ecp --help
./e2ecp serve --help
./e2ecp send --help
./e2ecp receive --help
```

## Project Structure

### Root Directory Files
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schollz/e2ecp](https://github.com/schollz/e2ecp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
