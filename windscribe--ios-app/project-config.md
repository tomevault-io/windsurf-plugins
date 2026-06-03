---
trigger: always_on
description: Operating notes for AI agents working in this repo. Humans want `README.md`.
---

# AGENTS.md

Operating notes for AI agents working in this repo. Humans want `README.md`.

## TL;DR

Windscribe iOS/tvOS VPN client. Multi-target Xcode project (no workspace, no Tuist). The codebase is **mid-migration**: UIKit+Combine+Swinject+Realm is the current shore, SwiftUI+async/await+constructor-injection+GRDB is the far shore. **When you touch existing code, match its style. When you write new code, steer toward the target state** — see [Modernization direction](#modernization-direction) below.

Do not mass-migrate existing screens. The UI is being rebuilt screen-by-screen; out-of-band rewrites create merge pain.

## Project Neo / v5.0 — AI agent onramp

If you're picking up Project Neo work, **read these in order before doing anything else**:

1. [`docs/PROJECT_NEO.md`](docs/PROJECT_NEO.md) — the rule sheet (non-negotiable rules for new code, branching strategy, "adding a new feature" walkthrough).
2. [`docs/neo/DECISIONS.md`](docs/neo/DECISIONS.md) — the architectural decision log. Every decision the team has settled on, with rationale.
3. [`docs/neo/STATE.md`](docs/neo/STATE.md) — the execution log. Read from the bottom to answer "where are we right now?".
4. [`docs/neo/plans/00-roadmap.md`](docs/neo/plans/00-roadmap.md) — the umbrella roadmap (M0–M10).
5. The relevant per-issue plan in [`docs/neo/plans/M<N>-*.md`](docs/neo/plans/) for the issue you're touching.

**Decisions and per-issue plans belong in tree, not in any agent's private memory.** Project Neo is multi-contributor and multi-agent; each teammate runs their own AI assistant with its own private memory. The only context every agent shares is what's in the repo. If you settled an architectural question or shipped a milestone, write it down in `DECISIONS.md` / `STATE.md` as part of the same PR — not in chat history, not in agent memory.

## Repository layout

```
iosapp/
├── Windscribe.xcodeproj/            # Single project, all targets live here
├── Windscribe/                      # Main iOS app target
│   ├── AppDelegate.swift            # @main, resolves services via Assembler.resolve
│   ├── SceneDelegate.swift
│   ├── Dependencies/                # Swinject DI wiring
│   │   ├── Assembler.swift          #   static `resolve<T>()` entry point
│   │   ├── CoreModule.swift         #   core services shared with extensions
│   │   ├── UserScope.swift          #   per-user (login-scoped) services
│   │   ├── WireguardModule.swift
│   │   └── AppModules/iOS/          #   iOS-specific registrations (VMs, routers, VCs)
│   ├── Router/                      # UIKit coordinator-style navigation
│   │   └── NavigationRouter/        #   SwiftUI navigation bridge (newer screens)
│   ├── ViewControllers/             # Legacy UIKit screens — MainViewController is the hub
│   ├── View/CustomViews/            # Shared UIKit subclasses (WSView, WSButton, etc.)
│   ├── Modules/                     # Newer, module-scoped features (SwiftUI-forward)
│   │   ├── Authentication/          #   Welcome, Login, SignUp, EmergencyConnect (SwiftUI + Combine VMs)
│   │   ├── News Feed/
│   │   ├── PlanUpgrade/
│   │   ├── Popup/
│   │   └── Preferences/             #   Settings surface, SwiftUI
│   ├── Managers/                    # Cross-cutting services (VPN, Session, Latency, …)
│   │   └── Protocols/               #   Manager protocols — inject these, not impls
│   ├── Repository/                  # Feature-scoped state/logic (one dir per domain)
│   ├── API/                         # Wrapper around the WSNet xcframework
│   │   ├── APIManager*.swift        #   public-facing async API surface
│   │   └── WSNet Protocol/          #   low-level wsnet bindings
│   ├── Data/
│   │   ├── Database/                #   Realm-backed LocalDatabase — target is GRDB
│   │   ├── FileDatabase/            #   on-disk blobs (server lists, logs)
│   │   ├── KeyChainDatabase/        #   credentials, session keys
│   │   └── Preferences/             #   UserDefaults wrapper (shared app group)
│   ├── Models/                      # Plain + Realm object models (mixed)
│   ├── Constants/                   # AppConstants, UIConstants, VPNProtocols, Enums, etc.
│   ├── Environments/
│   │   ├── Config.xcconfig          #   team ID, bundle IDs (override locally, don't commit)
│   │   └── Plists/                  #   Info.plist variants per scheme
│   └── Supporting Files/            # Assets, fonts, localization, Core Data model
├── WindscribeTV/                    # tvOS target — separate UI, shares Managers/Repo/API
├── PacketTunnel/                    # Network Extension: OpenVPN + IKEv2 provider
├── WireGuardTunnel/                 # Network Extension: WireGuard + AmneziaWG provider
│   └── Custom/                      #   amneziawg glue, x25519, bridging header
├── SiriIntents/                     # Legacy SiriKit intent handlers (Intents.framework)
├── AppIntents/                      # Modern AppIntents (iOS 16+) — Shortcuts entry points
├── HomeWidget/                      # WidgetKit extension (SwiftUI)
├── WindscribeTests/                 # XCTest unit tests + Mocks
├── libs/                            # Vendored xcframeworks
│   ├── WSNet.xcframework            #   core networking, written in C++ (scapix-bridged)
│   └── Proxy.xcframework

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Windscribe/iOS-App](https://github.com/Windscribe/iOS-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
