---
trigger: always_on
description: Animated desktop pet that reacts to Claude Code activity. A transparent, always-on-top Electron overlay (96×96px, bottom-right corner) shows a sprite-animated dog that responds to hook events.
---

# Code Pet

Animated desktop pet that reacts to Claude Code activity. A transparent, always-on-top Electron overlay (96×96px, bottom-right corner) shows a sprite-animated dog that responds to hook events.

## Tech Stack

- **Node.js** (>= 18) — hook scripts, process management
- **Electron** (^42.0.0) — transparent overlay window, the only runtime dependency
- No other external dependencies. Keep it that way.

## Directory Structure

```
.claude-plugin/plugin.json   # Claude Code plugin manifest
.claude-plugin/marketplace.json # Claude Code marketplace metadata
hooks/
  hooks.json                 # Hook event → script mapping
  scripts/                   # Hook handlers (plain Node.js, no Electron)
    bootstrap.js             # Lazy Electron installer (background npm install)
    send-event.js            # HTTP POST client to event server
    on-session-start.js      # SessionStart: bootstrap → launch app → send awaken
    on-session-end.js        # SessionEnd: send falling_asleep → shut down Electron
    on-notification.js       # Notification: send action_requested (+notification payload)
    on-prompt-submit.js      # UserPromptSubmit: send working_started or planning_started (+prompt_length)
    on-post-tool-use.js      # PostToolUse: sends action_completed for all tool completions
    on-stop.js               # Stop: send work_finished (+stop_reason)
src/
  app/                       # Electron main process
    main.js                  # Entry point: PID → server → overlay window
    event-server.js          # HTTP server on 127.0.0.1:31425 (/event, /health, /last-event, /shutdown)
    pet-registry.js          # PetRegistry class: per-project PetContext container with lifecycle callbacks
    pet-catalog.js           # Scans shipped + downloaded pet manifests, merges by id
    process-manager.js       # PID file, app launch/stop, health checks
    window-manager.js        # Transparent click-through BrowserWindow + marketplace IPC handlers
    logger.js                # File logger (~/.code-pet/code-pet.log, 1MB max)
    preload.js               # Context bridge: window.codePet.onPetEvent()
    settings-preload.js      # Context bridge for settings window (includes marketplace IPC)
    settings-store.js        # Persistent user settings (~/.code-pet/settings.json) with sound + dismissed pets
    terminal-focus.js        # macOS helper: focuses the terminal that spawned the session
    http-client.js           # Promise-based HTTP utility (Node.js built-in https/http, zero deps)
    marketplace-api.js       # Real marketplace REST API client (replaces MockLicenseAPI when configured)
    marketplace-catalog.js   # Catalog fetch + productId↔petId mapping (cached to product-map.json)
    marketplace-config.js    # Reads ~/.code-pet/marketplace.json for API URL, key, marketplace ID
    marketplace-constants.js # DEFAULT_BASE_URL, DEFAULT_MARKETPLACE_ID; consulted when no override
    license-api.js           # MockLicenseAPI (dev/test fallback when MARKETPLACE_MOCK=true)
    license-manager.js       # License activation, revalidation, offline grace period
    premium-store.js         # Downloads premium pet manifests + sprites to ~/.code-pet/pets/{id}/
    state-machine/             # Server-side state machine (whitelist pattern)
      states.js                # STATES enum
      events.js                # EVENTS, EVENT_TO_STATE, VALID_EVENTS
      base-state.js            # BaseState: ignore-all defaults, helpers
      state-factory.js         # createState(): state name → class instance
      pet-context.js           # PetContext: orchestrator, mutable state per project
      idle-state.js            # IdleState
      active-state.js          # ActiveState: shared working/planning base
      working-state.js         # WorkingState (extends ActiveState)
      planning-state.js        # PlanningState (extends ActiveState)
      waiting-for-action-state.js # WaitingForActionState
  renderer/                  # Chromium renderer (the visible overlay)
    index.html               # Shell: <div id="pets-container">, loads pet.js + pet-manager.js + ipc.js
    pet.js                   # Sprite state machine + interaction (Pet class)
    pet-manager.js           # Multi-project pet orchestration (PetManager class)
    pet-styles.js            # Builds + injects per-pet @keyframes/.pet.<state> CSS at runtime
    ipc.js                   # Wires IPC events to state machine
    styles.css               # Base/static styles for .pet (size, transitions); sprite rules are injected by pet-styles.js
    settings.html            # Settings window UI (opened on double-click)
    settings.js              # Settings window logic
    settings.css             # Settings window styling
    tabs/                    # Settings window tab partials
      general.html             # General settings tab
      store.html               # Marketplace / store tab
      usage.html               # Usage analytics tab
  tracking/                  # Skill / MCP tool usage tracking (self-contained)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mradovic95/code-pet](https://github.com/mradovic95/code-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
