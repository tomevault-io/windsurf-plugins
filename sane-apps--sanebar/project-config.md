---
trigger: always_on
description: > Project-specific settings that override/extend the global ~/CLAUDE.md
---

# SaneBar Project Configuration

> Project-specific settings that override/extend the global ~/CLAUDE.md

---

## Project Location

| Path | Description |
|------|-------------|
| **This project** | `~/SaneApps/apps/SaneBar/` |
| **Save outputs** | `~/SaneApps/apps/SaneBar/outputs/` |
| **Screenshots** | `~/Desktop/Screenshots/` (label with project prefix) |
| **Shared UI** | `~/SaneApps/infra/SaneUI/` |
| **Hooks/tooling** | `~/SaneApps/infra/SaneProcess/` |

**Sister apps:** SaneClip, SaneVideo, SaneSync, SaneHosts, SaneAI, SaneClick, SaneSales, Sane Mac Utilities Bundle

---

## Key Documentation

| Document | When to Use |
|----------|-------------|
| [GitHub Issues](https://github.com/sane-apps/SaneBar/issues) | Bug reports and tracking |
| [DEVELOPMENT.md](DEVELOPMENT.md) | Full SOP, 12 rules, compliance |
| [.claude/rules/](.claude/rules/README.md) | Code style rules by file type |
| [docs/DEBUGGING_MENU_BAR_INTERACTIONS.md](docs/DEBUGGING_MENU_BAR_INTERACTIONS.md) | Positioning bugs, coordinate system |

---

## Sane Philosophy

```
┌─────────────────────────────────────────────────────┐
│           BEFORE YOU SHIP, ASK:                     │
│                                                     │
│  1. Does this REDUCE fear or create it?             │
│  2. Power: Does user have control?                  │
│  3. Love: Does this help people?                    │
│  4. Sound Mind: Is this clear and calm?             │
│                                                     │
│  Grandma test: Would her life be better?            │
│                                                     │
│  "Not fear, but power, love, sound mind"            │
│  — 2 Timothy 1:7                                    │
└─────────────────────────────────────────────────────┘
```

→ Full philosophy: `~/SaneApps/meta/Brand/NORTH_STAR.md`

---

## PRIME DIRECTIVE (from ~/CLAUDE.md)

> When hooks fire: **READ THE MESSAGE FIRST**. The answer is in the prompt/hook/memory/SOP.
> Stop guessing. Start reading.



## Project Structure

| Path | Purpose |
|------|---------|
| `scripts/SaneMaster.rb` | Build tool - use instead of raw xcodebuild |
| `Core/` | Foundation types, Managers, Services |
| `Core/Services/` | Accessibility API wrappers, permission handling |
| `Core/Models/` | Data models (StatusItemModel, etc.) |
| `UI/` | SwiftUI views |
| `UI/Onboarding/` | Permission request flow |
| `Tests/` | Unit tests (regression tests go here) |
| `project.yml` | XcodeGen configuration |

---

## Build Strategy (PREFER MAC MINI)

SaneBar has local build issues on the MacBook Air. **Prefer the Mac Mini build server when reachable.**

```bash
# Check if mini is reachable (home network only — won't work from coffee shops)
ssh -o ConnectTimeout=3 mini 'echo ok' 2>/dev/null && echo "MINI AVAILABLE" || echo "MINI OFFLINE — use local"
```

- **Mini reachable** → use `mini-build.sh` for builds, `mini-test-install.sh` for DMG verification
- **Mini offline** → fall back to local `SaneMaster.rb` commands below

---

## Quick Commands

```bash
# Build & Test (local fallback)
./scripts/SaneMaster.rb verify          # Build + unit tests
./scripts/SaneMaster.rb test_mode       # Kill -> Build -> Launch -> Logs
./scripts/SaneMaster.rb logs --follow   # Stream live logs
./scripts/SaneMaster.rb verify_api X    # Check if API exists in SDK

# Release & App Store
./scripts/SaneMaster.rb release_preflight   # 9 pre-release safety checks
./scripts/SaneMaster.rb appstore_preflight  # App Store submission compliance

# CI/CD Helpers
./scripts/SaneMaster.rb enable_ci_tests   # Enable test targets for CI
./scripts/SaneMaster.rb restore_ci_tests  # Restore project.yml from CI backup
./scripts/SaneMaster.rb fix_mocks         # Add @testable import to mocks
./scripts/SaneMaster.rb monitor_tests     # Run tests with timeout + progress
./scripts/SaneMaster.rb image_info <path> # Extract image info and base64

# Memory Health (MCP Knowledge Graph)
./scripts/SaneMaster.rb mh              # Check entity/token counts
./scripts/SaneMaster.rb mcompact        # Compact verbose entities
./scripts/SaneMaster.rb mcleanup        # Generate MCP cleanup commands

# Circuit Breaker (Failure Tracking)
./scripts/SaneMaster.rb breaker_status  # Check if breaker is OPEN/CLOSED
./scripts/SaneMaster.rb breaker_errors  # Show failure messages
./scripts/SaneMaster.rb reset_breaker   # Reset after investigation

# Session Management
./scripts/SaneMaster.rb session_end     # End session + memory prompt
./scripts/SaneMaster.rb compliance      # Show session compliance report
```

---

## ⚠️ NSStatusItem Positioning - POSITION PRE-SEEDING

**SaneBar uses position pre-seeding for NSStatusItem ordering, but that is only the first layer.**

→ Start with `docs/MENU_BAR_RUNTIME_PLAYBOOK.md`
→ Lower-level positioning notes: `docs/DEBUGGING_MENU_BAR_INTERACTIONS.md`

### The Pattern (10 lines, battle-tested)

```swift
// 1. SEED ordinal positions in UserDefaults BEFORE creating items
private static func seedPositionsIfNeeded() {
    let defaults = UserDefaults.standard
    if defaults.object(forKey: "NSStatusItem Preferred Position \(mainAutosaveName)") == nil {
        defaults.set(0, forKey: "NSStatusItem Preferred Position \(mainAutosaveName)")  // 0 = rightmost
    }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sane-apps/SaneBar](https://github.com/sane-apps/SaneBar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
