---
trigger: always_on
description: Rilable is an open-source iOS app that builds **live web apps and native iOS apps from a
---

# Rilable — agent setup guide

Rilable is an open-source iOS app that builds **live web apps and native iOS apps from a
prompt** — a Lovable/Replit-mobile-style builder you run on your own accounts. SwiftUI front
end, Convex backend, Claude for code generation, Daytona cloud sandboxes for web apps, Chorus
cloud Xcode builds for iOS apps.

**If you are an AI coding agent (Claude Code etc.): your job is to get this running for the
user end-to-end.** Drive every step below yourself — run the commands, ask the user for keys at
the moments they're needed (with the exact URLs to get them), verify each stage before moving
on, and finish with the app running in the iOS Simulator. The user should never have to read
documentation.

## Architecture (30 seconds)

```
iOS app (SwiftUI, ios/)  ── convex-swift websocket ──►  Convex backend (backend/convex/)
  the user types a prompt          projects/messages/files tables, real-time
                                   "builder" node action orchestrates:
                                   ├─ Anthropic API — Claude writes the app code
                                   ├─ [web]    Daytona REST — public sandbox serves it
                                   ├─ [mobile] Chorus REST — cloud Xcode build + browser
                                   │           iPhone-simulator preview + OTA device installs
                                   ├─ OpenAI Whisper — voice input (optional)
                                   └─ Vercel AI Gateway proxy (convex/http.ts) — gives every
                                      generated app free AI features (optional)
```

## Prerequisites — verify before anything else

```bash
xcodebuild -version   # Xcode 16+ on macOS
node -v               # Node 18+
xcodegen --version    # brew install xcodegen
```

An iPhone Simulator (any iOS 17+ runtime) is enough; a physical iPhone is optional.

## Step 1 — Convex backend (free)

```bash
cd backend && npm install
npx convex dev --once --configure new
```

This opens a browser login (GitHub/Google, free tier is fine), creates a project, pushes the
functions, and writes `backend/.env.local` containing `CONVEX_URL` — you'll need that URL in
Step 3.

## Step 2 — API keys

Ask the user for these one at a time, then set each with
`npx convex env set NAME value` (run inside `backend/`):

| Env var | Get it at | Powers | Needed for |
|---|---|---|---|
| `ANTHROPIC_API_KEY` | console.anthropic.com → Settings → API Keys | Claude writes the apps | **Required** |
| `DAYTONA_API_KEY` | app.daytona.io → Settings → API Keys | web-app sandboxes | Web builds |
| `CHORUS_API_KEY` | ios.chorus.com (Vibecode's iOS build service — install its CLI with `curl -fsSL https://ios.chorus.com/install.sh \| bash`, then `./build-ios-apps/ios-cli login`; the key starts with `chorus_`) | cloud Xcode builds + signing | Mobile builds |
| `CHORUS_USER_ID` | printed by the Chorus CLI after login (also in `~/.vibecode/ios/config.json` as `userId`) | device-install signing | Mobile installs |
| `OPENAI_API_KEY` | platform.openai.com → API keys | Whisper voice input | Optional |
| `VERCEL_AI_GATEWAY_KEY` | vercel.com dashboard → AI Gateway → API keys (starts `vck_`) | AI features inside generated apps | Optional |

Minimum viable setup: `ANTHROPIC_API_KEY` + `DAYTONA_API_KEY` (web builds only). Skip any
optional key the user doesn't want — the related feature degrades gracefully.

## Step 3 — iOS app

```bash
cp ios/AppConfig.example.swift ios/Sources/AppConfig.swift
# edit ios/Sources/AppConfig.swift:
#   convexDeploymentURL = the CONVEX_URL from backend/.env.local
#   userName            = the user's first name (home-screen greeting)
```

Optionally edit `ios/project.yml`: set `PRODUCT_BUNDLE_IDENTIFIER` to a reverse-DNS id the
user owns and `DEVELOPMENT_TEAM` to their Apple team id (only needed for running on a physical
iPhone — simulator builds work with the defaults).

```bash
cd ios
xcodegen generate
xcodebuild -project Forge.xcodeproj -scheme Forge \
  -destination 'platform=iOS Simulator,name=iPhone 17' \
  -derivedDataPath build CODE_SIGNING_ALLOWED=NO build
xcrun simctl boot "iPhone 17" 2>/dev/null; open -a Simulator
xcrun simctl install booted build/Build/Products/Debug-iphonesimulator/Forge.app
xcrun simctl launch booted $(defaults read "$(pwd)/build/Build/Products/Debug-iphonesimulator/Forge.app/Info" CFBundleIdentifier)
```

(Substitute any installed simulator name. If voice input will be used in the simulator:
`xcrun simctl privacy booted grant microphone <bundle-id>`.)

## Step 4 — verify end-to-end

Backend smoke test (web build, ~2 min):

```bash
cd backend
ID=$(npx convex run projects:create '{"prompt":"a hello world page with one big button"}' | tr -d '"')
# poll until "live", then curl the previewUrl it returns:
npx convex run projects:get "{\"id\":\"$ID\"}"
```

Then in the app: tap **Enter App**, type a prompt, watch it build, tap the build card's
**Preview**. For mobile builds flip the **Web | Mobile** toggle (needs the Chorus key; first
device install will walk the user through Apple sign-in via a login link in the chat).

## Things agents commonly get wrong here

- After adding ANY new Swift file you must re-run `xcodegen generate` — the project uses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rbrown101010/rilable](https://github.com/rbrown101010/rilable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
