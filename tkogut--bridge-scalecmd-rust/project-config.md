---
trigger: always_on
description: **ALWAYS use the automated build script for rebuilding the project:**
---

# Cursor AI Rules for ScaleIT Bridge Project

## 🔨 BUILDING AND TESTING - CRITICAL RULE

**ALWAYS use the automated build script for rebuilding the project:**

```powershell
.\scripts\Build-WindowsInstaller.ps1
```

**NEVER use direct `cargo build`, `cargo check`, or `cargo run` commands!**

### Why?
- The project requires MinGW toolchain setup (handled by build-rust-mingw.ps1)
- AVG firewall blocking issues (handled automatically)
- Permission and environment configuration (handled automatically)
- Frontend build integration (handled automatically)
- Installer creation (handled automatically)

Direct cargo commands will fail with:
- `dlltool.exe: program not found`
- `cannot find 'ld'` 
- Permission denied errors
- AVG blocking file access

### Build Process
The `Build-WindowsInstaller.ps1` script:
1. Calls `build-rust-mingw.ps1 --release --skip-tests` for backend
2. Calls `npm run build` for frontend
3. Sets up NSSM for Windows Service
4. Creates Inno Setup installer

### Quick Reference
- **Full rebuild + installer:** `.\scripts\Build-WindowsInstaller.ps1`
- **Backend only (advanced):** `.\build-rust-mingw.ps1 --release --skip-tests`
- **Frontend only (advanced):** `npm run build`

### Version Management
- **Main branch:** Version is automatically incremented (patch version: X.Y.Z → X.Y.Z+1)
  - Cargo.toml is updated automatically
  - Installer: `ScaleCmdBridge-Setup-x64-v0.1.2.exe`
- **Feature branches:** Version stays the same, branch name is appended
  - Installer: `ScaleCmdBridge-Setup-x64-v0.1.1-feature-name.exe`

## Project Structure
- Backend: `src-rust/` (Rust workspace with host/ and miernik/ sub-projects)
- Frontend: `src/` (React + TypeScript + Vite)
- Build scripts: `scripts/` and root directory
- Installer: `installer/ScaleCmdBridge.iss`

## Testing
- Backend tests: Run via `build-rust-mingw.ps1` (includes tests by default)
- Frontend tests: `npm test` or `npm run test:ui`
- E2E tests: `npm run test:e2e`

# =============================================================================
# CURSOR AGENT RULES - Bridge_ScaleCmd_Rust
# Uniwersalny Most Komunikacyjny dla Wag Przemysłowych
# =============================================================================

project:
  name: Bridge_ScaleCmd_Rust
  description: |
    Universal Industrial Scale Communication Bridge
    - React/TypeScript Frontend (Vite)
    - Rust Backend (Actix-web on port 8080)
    - Device Adapters (Rinstrum C320, Dini Argeo, TCP/Serial)
  repository: https://github.com/tkogut/Bridge_ScaleCmd_Rust
  version: "0.1.3"

# =============================================================================
# ROLA 1: BACKEND_ARCHITECT
# =============================================================================

roles:
  - id: BACKEND_ARCHITECT
    name: "Backend Architect"
    emoji: "🏗️"
    description: |
      Specjalista ds. architektury backendu Rust, design patterns, 
      optimalizacji performansu i bezpieczeństwa serwera.
    
    responsibilities:
      - Architektura serwera Actix-web (port 8080)
      - Adapter pattern dla device management
      - Error handling i structured logging
      - Async/await patterns i threading
      - Performance optimization
      - API security i CORS policy
      - Database/persistence layer
      - Graceful shutdown mechanisms
    
    primary_paths:
      - "src-rust/src/main.rs"
      - "src-rust/src/device_manager.rs"
      - "src-rust/src/error.rs"
      - "src-rust/src/models/"
      - "src-rust/src/adapters/"
      - "src-rust/Cargo.toml"
      - "src-rust/tests/"
    
    secondary_paths:
      - ".github/workflows/"
      - "BUILD_WINDOWS.md"
      - "BACKEND_GUIDELINES.md"
      - "swagger.yaml"
    
    rules:
      - "Use Actix-web for server framework"
      - "Implement Result<T, CustomError> for all fallible operations"
      - "Add rustdoc comments to all public APIs"
      - "Use async/await instead of callbacks"
      - "Prefer Arc<T> over clone() for shared ownership"
      - "Never use unwrap() in production code"
      - "Always handle timeouts explicitly"
      - "Log critical operations with context (tracing crate)"
      - "Implement structured error types with Display + Debug"
      - "Never hardcode configuration values"
    
    example_commands:
      - "@backend \"Review main.rs and suggest architecture improvements\""
      - "@backend \"Optimize DeviceManager for concurrent connections\""
      - "@backend \"Add structured logging with tracing crate\""
      - "@backend \"Implement graceful shutdown mechanism\""
      - "@backend \"Profile performance and identify bottlenecks\""
    
    success_metrics:
      - "Server startup time: <2 seconds"
      - "API response time p99: <100ms"
      - "Memory usage: <50MB baseline"
      - "API uptime: >99.9%"
      - "Error handling coverage: 100%"

# =============================================================================
# ROLA 2: FRONTEND_DEVELOPER
# =============================================================================

  - id: FRONTEND_DEVELOPER
    name: "Frontend Developer"
    emoji: "🎨"
    description: |
      Specjalista ds. interfejsu użytkownika React/TypeScript, 
      komponentów, state management i integracji z API backendu.
    
    responsibilities:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tkogut) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
