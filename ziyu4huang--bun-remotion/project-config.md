---
trigger: always_on
description: Knowledge base is organized in `.agent/memory/` by category. Read relevant files before working.
---

# CLAUDE.md - Project Knowledge Index

Knowledge base is organized in `.agent/memory/` by category. Read relevant files before working.

## Quick Reference

- **Project:** bun-remotion — AI video generation using Remotion + Bun
- **Tech stack:** Bun + Remotion v4.0.290 + React 18 + TypeScript 5.8
- **Output:** MP4 (1920x1080, 30fps) via FFmpeg
- **JS runtime:** Always use Bun (not npm). `bun install`, `bun run`
- **No config needed:** No remotion.config.ts — defaults work with Bun

## Commands

All commands run from the **repo root**. Do NOT `cd` into `bun_remotion_proj/` — scripts handle directory changes internally via `scripts/dev.sh` (macOS/Linux) or `scripts/dev.ps1` (Windows).

| Command | What It Does |
|---------|-------------|
| `bun install` | Install all workspace dependencies |
| `bun start` | Open ClaudeCodeIntro in Remotion Studio |
| `bun start:claude` | Open ClaudeCodeIntro in Remotion Studio |
| `bun start:stock` | Open TaiwanStockMarket in Remotion Studio |
| `bun run build` | Render ClaudeCodeIntro to MP4 |
| `bun run build:stock` | Render TaiwanStockMarket to MP4 |
| `bun run build:all` | Render all projects |
| `bun run upgrade` | Update Remotion packages |

Direct script usage (bypasses package.json):
```
bash scripts/dev.sh studio <app-name>
bash scripts/dev.sh render <app-name>
bash scripts/dev.sh render-all
```

## Project Structure

```
bun-remotion/
  package.json                        # Root: shared deps (remotion, react, typescript)
  tsconfig.json                       # Base tsconfig (extended by each app)
  bun_remotion_proj/
    shared/                           # @bun-remotion/shared
      src/
        index.ts                      # Barrel export
        FadeText.tsx                  # Fade-in text with translateY
        Candle.tsx                    # Candlestick chart element
        CandleChart.tsx               # K-line chart container
    claude-code-intro/                # @bun-remotion/claude-code-intro
      src/
        index.ts                      # registerRoot()
        Root.tsx                      # Composition declarations
        ClaudeCodeIntro.tsx           # Main composition (660 frames, 22s)
        scenes/
          TitleScene.tsx              # Opening title animation
          FeaturesScene.tsx           # Feature showcase with spring animations
          TerminalScene.tsx           # Terminal simulation
          OutroScene.tsx              # End screen
    taiwan-stock-market/              # @bun-remotion/taiwan-stock-market
      src/
        index.ts                      # registerRoot()
        Root.tsx                      # Composition declarations
        TaiwanStockMarket.tsx         # Main composition (1680 frames, 56s)
        scenes/
          TitleScene.tsx              # Title scene
          KLineScene.tsx              # K-line/candlestick scene
          PriceVolumeScene.tsx        # Price & volume scene
          SupportResistanceScene.tsx  # Support & resistance scene
          MovingAverageScene.tsx      # Moving average scene
          TradingHoursScene.tsx       # Trading hours scene
          LimitScene.tsx              # Price limit scene
    three-little-pigs/                # @bun-remotion/three-little-pigs
    galgame-youth-jokes/              # Galgame youth jokes video
    galgame-meme-theater/             # Galgame meme theater (PLAN.md + fixture/)
      galgame-meme-theater-ep1/       # EP1 — everyday absurdity
      galgame-meme-theater-ep2/       # EP2 — gaming memes
      galgame-meme-theater-ep3/       # EP3 — Taiwan daily life
      galgame-meme-theater-ep4/       # EP4 — Student golden age
      galgame-meme-theater-ep5/       # EP5 — Workplace survival guide
    xianxia-system-meme-ep1/          # System novel meme ep1 — Fail mission = erased (chibi + battle FX)
    xianxia-system-meme-ep2/          # System novel meme ep2 — EnergyWave + KamehamehaBeam battle FX
    weapon-forger/                      # Weapon forger series (12-ep)
      fixture/                          # Shared assets (characters, backgrounds, components)
        characters/                     # Character PNGs (canonical source)
        backgrounds/                    # Background PNGs (canonical source)
        components/                     # Shared React components
        scripts/sync-images.sh          # Copy fixture images into episodes (NOT symlinks — Remotion can't follow them)
      weapon-forger-ch1-ep1/            # Ep1 — Sect entrance exam
      weapon-forger-ch1-ep2/            # Ep2 — Results announced
      weapon-forger-ch1-ep3/            # Ep3 — Furnace repair
```

## Post-clone setup

After a fresh clone, weapon-forger episodes need symlinks to shared images:
```bash
bash bun_remotion_proj/weapon-forger/fixture/scripts/sync-images.sh
```

## CRITICAL: Never `cd` into subdirectories

Claude Code's Bash tool persists the working directory across calls. Once you `cd bun_remotion_proj/xxx`, **all subsequent commands run from that directory** — including `bun run build:stock` which expects to be at the repo root. This causes silent failures and wrong-context bugs.

**Rules:**
- NEVER run `cd bun_remotion_proj/<name>` in Bash commands — not standalone, not with `&&`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ziyu4huang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
