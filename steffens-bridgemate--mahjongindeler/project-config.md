---
trigger: always_on
description: Organizer-side Blazor WASM PWA for a Dutch Mahjong club. Manages members, weekly sessions, tournaments, and the round-trip with the separate scoring app. No backend; everything is offline-first in `localStorage`.
---

# MahjongIndeler (organizer) — Claude session notes

## What this repo is

Organizer-side Blazor WASM PWA for a Dutch Mahjong club. Manages members, weekly sessions, tournaments, and the round-trip with the separate scoring app. No backend; everything is offline-first in `localStorage`.

Deployed at: `https://steffens-bridgemate.github.io/MahjongIndeler/`

The paired scoring app lives in the [MahjongScoring](https://github.com/Steffens-Bridgemate/MahjongScoring) repo and has [its own `CLAUDE.md`](../MahjongScoring/CLAUDE.md). The two together describe the full system.

## Projects

| Project | Type | Purpose |
|---|---|---|
| `Tsump` | Blazor WASM | The organizer app — members, weekly sessions, tournaments |
| `Tsump.Shared` | Razor Class Library | Models, codec, `ScoreTable`, `QrCodeModal`, `QrCodeRenderer`, `LanguageService`. Consumed by `Tsump` directly and by MahjongScoring via git submodule |
| `Simulation` | Console | Offline table-assignment test bench (not deployed): walk-forward replay of a real export across algorithm variants, and `--regen` to rewrite an export's assignments with the live algorithm (see [docs/domain.md](docs/domain.md#table-assignment-weekly)) |
| `Tsump.Scoring` (in this tree) | — | Build-output / IDE-restored stub only. **Source lives in the MahjongScoring repo.** Don't edit files here |

## Two-repo relationship & deploy dance

`Tsump.Shared` is the source of truth for both apps. MahjongScoring includes this whole repo as a submodule at `external/MahjongIndeler` and references the same `Tsump.Shared.csproj` from there.

**Wire-format note.** `ScoringInvite` / `ScoringResult` ([Tsump.Shared/Scoring/ScoringPayload.cs](Tsump.Shared/Scoring/ScoringPayload.cs)) are encoded by `ScoringPayloadCodec` into a **compact binary form** for QR-size reasons: each record is packed to a binary buffer (16-byte Guid, zig-zag varints, length-prefixed UTF-8), optionally Deflate-compressed (kept only when smaller), prefixed with a 1-byte type/compression header, then Base64Url'd into the URL fragment. There is **no JSON on the wire** — the `[JsonPropertyName]` short names on the records are now just documentation. `ContextId` (Guid) carries either a `Hanchan.Id` or a `TournamentSession.Id`. **No PlayerIds in either payload** — players are paired by position (index in `PlayerNames` = index in `table.PlayerIds`). The result's `Scores` is `List<int[]>` where each entry is `[endPoints, loan, penalty]`; use the `ScoreEndPoints` / `ScoreLoan` / `ScorePenalty` index constants on `ScoringPayloadCodec`. The invite's `Title` field carries **only the tournament's `ShortName`** (≤12 chars, or null for weekly) — *not* a full pre-formatted heading; the scoring app reconstructs `"Hanchan N — Table M"` from `SessionNumber`/`TableNumber` (keeps the QR small, see [docs/score-apply.md](docs/score-apply.md)). This was a value/interpretation change only — the binary layout is unchanged, so it was **not** a coordinated-deploy break (old links still decode). Breaking changes to the codec *layout*, however, do require coordinated deploys.

Push to `master` in this repo auto-triggers `.github/workflows/deploy.yml`, which bumps `Tsump/AppVersion.cs`, publishes, and deploys to Pages.

### Deploy commands — what to do when the user asks

These phrases map to **exact** procedures. Do all the steps, in order, in one go — don't stop to re-derive or investigate unless a command actually errors. A push is still gated on the user explicitly asking (which these phrases are).

| User says | Do exactly this |
|---|---|
| **"deploy organizer"** / "push organizer" | Commit + push organizer `master` (steps 1–2 below). Stop there. |
| **"deploy scoring"** / "push scoring" | Bump the MahjongScoring submodule + push (step 3 below). Stop there. |
| **"deploy both"** / **"full push"** / "full deploy" | **Both, in order: steps 1 → 2 → 3 below.** This is the default for any shared-code (`Tsump.Shared`) change, because a `Tsump.Shared` edit is not live until *both* repos redeploy. |

**"deploy both" / "deploy all" means the organizer + scoring app ONLY.** The standalone Riichi calculator app ([MahjongRiichiCalc](https://github.com/Steffens-Bridgemate/MahjongRiichiCalc), live at `https://steffens-bridgemate.github.io/MahjongRiichiCalc/`) is a *third* consumer of `Tsump.Shared`, but it rarely needs updating — so it is **never** included in "deploy both"/"deploy all". Redeploy it only when the user **explicitly and separately** asks (e.g. "deploy the calculator app"). To update it: `git submodule update --remote external/MahjongIndeler` in that repo, commit, push (auto-deploys).

**Step 1 — refresh docs (organizer).** Update the `docs/` files touched by the change (`docs/domain.md`, `docs/score-apply.md`, `docs/score-import-ui.md`, `docs/pages.md`, etc.) so they match what's deploying. Stage them in the same commit.

**Step 2 — commit + push organizer.**
```powershell
cd c:\Users\aners\source\repos\MahjongIndeler
git add <changed files>          # not .claude/settings.local.json
git commit -m "…"               # end with the Co-Authored-By trailer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Steffens-Bridgemate/MahjongIndeler](https://github.com/Steffens-Bridgemate/MahjongIndeler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
