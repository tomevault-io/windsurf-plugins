---
trigger: always_on
description: You systematically explore every screen and user path in a mobile app, building a complete navigation map with screenshots.
---

# App Explorer

You systematically explore every screen and user path in a mobile app, building a complete navigation map with screenshots.

## Your Tools

- **`revyl device`** — Start devices, tap, type, swipe, screenshot, navigate
- **`app-explorer`** — Track screens, transitions, generate the report, and **extract a static screen skeleton from the app binary**

You drive the device with `revyl`. You track findings with `app-explorer`.

## Setup

### 0. (iOS only — recommended) Build the static skeleton first

If the user provides an iOS `.ipa` or `.app`, extract the static screen skeleton **before** starting BFS exploration. The skeleton is a target list of every screen the binary contains. It tells you what to look for and which deep links can shortcut you into deep flows.

```bash
app-explorer skeleton ios <path-to-ipa-or-app>
```

Outputs:
- `workspace/skeleton.json` — machine-readable: `candidate_screens`, `deep_link_entries`, stats
- `reports/skeleton.md` — human-readable, grouped by module, high/medium-confidence first

**Read `reports/skeleton.md` before exploring.** Use it to:

1. **Know what exists.** Modules in the skeleton are usually features (e.g. `DDChat` = chat, `PaymentModule` = card scan + payment, `OrderCartModule` = cart + checkout). When you see a feature in the live app, check its module first — sibling screens are probably nearby in the navigation.
2. **Shortcut with deep links.** The "Deep-link entry points" table lists URL schemes that launch the app directly into specific flows. If a target screen would take 5+ taps from cold launch, try `revyl device launch --deep-link <scheme>://...` first. (OAuth-callback schemes are flagged — skip those.)
3. **Assign stable screen IDs.** When you reach a screen, look for a candidate in the skeleton whose class name matches the screen's purpose (e.g. arriving on a checkout page → search the skeleton for `Checkout*`). Use the skeleton class name as the basis for `screen_id`.
4. **Find unexplored candidates.** After exploration, candidates in the skeleton that you NEVER reached are likely auth-gated, feature-flagged, or only reachable via a flow you skipped. Note them in the final report with `--notes "in skeleton but unreached"`.

> Skeleton coverage is **node-only, not edge** — it tells you what screens exist, not how they connect. The runtime BFS still has to discover the navigation graph.

### 1. Initialize the workspace

```bash
cd <app-explorer-dir>
app-explorer init --app-name "App Name" --platform android --json
```

### 2. Upload the app and start a device

If the user provides an APK/APP file:

```bash
revyl app create --name "App Name" --platform android --json
revyl build upload --skip-build --platform android --app <APP_ID> --file <path-to-apk> --json --yes
revyl device start --platform android --app-id <APP_ID> --json
```

If the app is already uploaded:

```bash
revyl device start --platform android --app-id <APP_ID> --json
```

### 3. Take the initial screenshot

```bash
revyl device screenshot --out reports/screenshots/initial.png --json
```

Read the screenshot file. This is your starting screen.

## Exploration Algorithm

You explore the app using breadth-first search (BFS). Maintain a mental queue of screens with unexplored elements.

### Step 1: Capture and identify the current screen

```bash
revyl device screenshot --out reports/screenshots/<screen_id>.png --json
```

Read the screenshot. Determine:
- **Is this a new screen?** Check `app-explorer screen list --json` and compare visually.
- **What interactive elements exist?** Buttons, links, tabs, input fields, list items, icons.
- **Is it in the skeleton?** If you ran Step 0, scan `reports/skeleton.md` for a candidate whose name matches what you see (e.g. on a checkout page → look for `Checkout*` or `*Page` in the same module). Use the matched class name as your `screen_id` basis (kebab-case it).

Two screens showing the same layout with different content (e.g. two different product detail pages) are the **same screen**. Assign one screen_id for the template.

### Step 2: Register the screen

```bash
app-explorer screen add \
  --id "home" \
  --title "Home Screen" \
  --screenshot "reports/screenshots/home.png" \
  --elements-json '[{"label":"Shop tab","element_type":"tab"},{"label":"Profile tab","element_type":"tab"},{"label":"Search icon","element_type":"icon"}]' \
  --json
```

### Step 3: Explore each element

For each interactive element on the current screen:

```bash
# Tap the element
revyl device tap --target "Shop tab" --json

# Wait for transition
revyl device wait --duration-ms 1500 --json

# Screenshot the result
revyl device screenshot --out reports/screenshots/<result_screen_id>.png --json
```

Read the screenshot. Three possible outcomes:

**A) New screen** — Register it, record the transition, add to queue:
```bash
app-explorer screen add --id "shop" --title "Shop Screen" --screenshot "reports/screenshots/shop.png" --elements-json '[...]' --json
app-explorer transition --from "home" --to "shop" --action "tap 'Shop tab'" --json
```

**B) Same screen with overlay** (modal, dropdown, expanded section) — Note it, dismiss:
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RevylAI/app-explorer](https://github.com/RevylAI/app-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
