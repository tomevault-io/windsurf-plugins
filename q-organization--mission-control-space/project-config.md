---
trigger: always_on
description: The developer primarily uses TTS (text-to-speech / voice dictation) to issue commands. Common misheard words to watch for:
---

# Mission Control Space

## Voice-to-Text Notice

The developer primarily uses TTS (text-to-speech / voice dictation) to issue commands. Common misheard words to watch for:

| You might hear | Actual meaning |
|----------------|----------------|
| Cloud, Claude | Claude (the AI) |
| model | modal |
| sub-base, subbase | Supabase |
| planet-form | platform |
| no-tion, ocean | Notion |
| way-socket | WebSocket |
| bite, white | Vite |
| super base | Supabase |
| hooks | hooks (React) |
| hurler | Howler (audio lib) |
| foul AI, fall AI, fal | FAL.ai (image generation API) |
| edge function | Supabase Edge Function |
| real time, real-time | Realtime (Supabase feature) |

Always interpret ambiguous words in the context of this project's tech stack before asking for clarification.

---

## What This Project Is

Mission Control Space is a **multiplayer productivity game** where players control spaceships in a 2D space world. The core purpose: players complete real tasks (synced from Notion) and business/product goals by flying to planets and landing on them. Everything is designed for **multiplayer** - the whole point of the game is that players can see each other's progress, evolution, and activity in real time.

### Core Loop
1. Players fly spaceships around a 10,000 x 10,000 pixel space world
2. Planets represent goals and tasks (business milestones, product features, Notion tickets)
3. Landing on a planet completes the task and earns points
4. Points are spent on ship upgrades (size, speed, weapons, visual effects)
5. All progress is visible to other players in real time

### Players & Zones
Each player has a dedicated zone in the world:
- **Quentin** (orange) - East (8000, 5000)
- **Alex** (blue) - Northeast (7100, 2900)
- **Armel** (green) - North (5000, 2000)
- **Solène** (pink) - Northwest (2900, 2900)
- **Hugues** (purple) - West (2000, 5000)

Notion tasks assigned to a player appear as planets in their zone.

---

## Tech Stack

- **Framework**: React 18 + TypeScript
- **Build**: Vite (dev server on port 5175)
- **Rendering**: HTML5 Canvas 2D (custom game engine, no framework)
- **Database & Realtime**: Supabase (PostgreSQL + Realtime subscriptions)
- **Multiplayer Positions**: WebSocket server (Node.js `ws` library)
- **Audio**: Howler.js
- **Animations**: canvas-confetti
- **AI Images**: FAL.ai (nano-banana model)
- **Deployment**: AWS S3 (frontend) + EC2 (WebSocket server)

---

## Available Tools — Use Them Proactively

You have full access to the following tools. **Proactively offer to use them** whenever they can help achieve a goal — don't wait to be asked.

### Supabase CLI
The Supabase CLI (`supabase`) is installed and available. Use it to:
- Create/modify database tables and columns
- Deploy and manage Edge Functions
- Run migrations
- Inspect database state
- Manage realtime publications

If a feature needs a new table, column, Edge Function, or RLS policy — just do it directly via the CLI.

### AWS Deployment (Automatic via GitHub Actions)
Pushing to `main` triggers a GitHub Actions workflow that automatically builds and deploys the frontend to AWS S3. You do NOT need to manually deploy. Just push the code and it ships.

### WebSocket Server Deployment (Manual via SSH)
The WebSocket server runs on EC2 (`13.250.26.247:8080`, Singapore `ap-southeast-1`). The SSH key is at the project root: `mission-control-ws.pem`. Deploy with:
```bash
scp -i mission-control-ws.pem ws-server/server.js ec2-user@13.250.26.247:~/ws-server/
ssh -i mission-control-ws.pem ec2-user@13.250.26.247 "cd ~/ws-server && pm2 restart mission-control-ws"
```
Deploy the WS server **before** pushing frontend changes that depend on new message types.

### FAL.ai Image Generation API
The FAL.ai API key is available in `src/App.tsx` (constant `FAL_API_KEY`). Use it to generate images for:
- New planet visuals
- Ship upgrade artwork
- Custom planet mascots
- Any visual asset the game needs

When working on a feature that could benefit from a generated image (new planet type, new upgrade, new visual element), **offer to generate one** using the FAL.ai API rather than using placeholder graphics.

### Summary
You have the database CLI, automatic frontend deployment, WebSocket server SSH access, and an image generation API. When implementing a feature, use the full toolkit end-to-end — create the DB schema, write the frontend code, deploy WS server changes if needed, generate any needed images, and the frontend deployment happens automatically on push.

---

## Global UI Rules

### No Visible Scrollbars

**Anywhere in the entire application**, if content is scrollable, the scrollbar must be **hidden visually** while still allowing scroll functionality. Apply this CSS pattern globally:

```css
/* Hide scrollbar but keep scroll functionality */
scrollbar-width: none;           /* Firefox */
-ms-overflow-style: none;        /* IE/Edge */
&::-webkit-scrollbar {           /* Chrome/Safari/Opera */
  display: none;
}
```

For inline styles in React:
```tsx
style={{
  overflow: 'auto',           // or 'scroll' — allow scrolling
  scrollbarWidth: 'none',     // Firefox
  msOverflowStyle: 'none',   // IE/Edge
}}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Q-organization) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
