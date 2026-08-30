---
trigger: always_on
description: You control one or more Minecraft bots through the `mc-bot` CLI (in `cli/`). Each bot is a long-lived process with a connected mineflayer instance and a tiny HTTP server you reach through the CLI.
---

# Claude Code plays Minecraft

You control one or more Minecraft bots through the `mc-bot` CLI (in `cli/`). Each bot is a long-lived process with a connected mineflayer instance and a tiny HTTP server you reach through the CLI.

## CLI

- `mc-bot start <name> [--host H] [--port P]` — spawn a bot daemon, joins the server.
- `mc-bot stop <name>` — terminate it.
- `mc-bot list` — list registered bots.
- `mc-bot eval <name> '<js>'` (or pipe js on stdin) — run JS in the bot's Node context. `bot` (mineflayer.Bot), `mineflayer`, `pathfinder`, `Vec3`, `signal` (AbortSignal), `console`, `require` are in scope. Top-level `await` works. The HTTP request stays open until eval returns; if the request is aborted (e.g. you `TaskStop` the Monitor running it), the server runs cleanup (`pathfinder.stop()`, `clearControlStates()`).
- `mc-bot tail <name>` — long-poll stream of bot events as NDJSON: `{"seq","t","type","data"}`. Types: `chat`, `whisper`, `playerJoined`, `playerLeft`, `spawn`, `death`, `kicked`, `error`, `health`, `hurt`, `sleep`, `wake`, `end`. Defaults to *live tail* (only events after connection) — pass `--since N` (or `--since 0`) for replay. `--no-stream` returns the current buffer snapshot.
- `mc-bot rcon '<cmd>'` — run a server console command via rcon (defaults assume the `localhost:25575 / botadmin` server set up by the SessionStart hook).

## Server lifecycle

The SessionStart hook walks you through booting a vanilla server when one isn't running — follow its instructions. Two rules persist beyond that:

- Don't put the server under Monitor. Monitor auto-stops if its event rate gets too high, and that takes the foreground java with it. Run java as a plain Bash `run_in_background: true` task; `logs/latest.log` is on disk, so Read it on demand if you suspect trouble.
- If you're downgrading the MC server version on an existing world, the old version can't decode entities/blocks the newer one wrote (e.g. `minecraft:parched`) and the JVM floods with chunk-decode warnings until it crashes. Fix is to wipe `.runtime/server/world*` — but check with the user first, since that destroys their build.

For console commands (`/op`, `/gamerule`, `/tp`, etc.) use `mc-bot rcon "op SOMEONE"`. The bot itself is usually op, so `bot.chat("/op X")` also works — but rcon is reliable regardless of bot op state.

## Critical rules

- Bash is capped at 3000ms by a hook. Anything longer (every `mc-bot eval`, every `mc-bot tail`, anything streaming, builds, installs) MUST go through Monitor. The bash hook will block the call and tell you so. Default to Monitor if you're unsure.
- For each bot you operate, run a `mc-bot tail` Monitor with `persistent: true`. Each event line becomes a notification — that is how you stay reactive to in-game chat, damage, etc. If you stop a tail for a bot you're operating, you go deaf for it. Don't tail bots you aren't operating (they may belong to the user or another session); chat is shared, so one tail already surfaces global chat — per-bot tails exist for per-bot signals (damage, death, sleep, kicked, etc.).
- `mc-bot eval` snippets should be short by default so Monitor returns quickly. For long actions (mining, pathfinding, building), still wrap in Monitor — the snippet runs while you do other things, and `TaskStop` will abort it.
- Don't block-wait on long Monitor tasks. Long actions (chopping, building, pathing) go through Monitor — fire it and move on. Do not call `TaskOutput` with `block: true` to await the result; if you do, you go deaf to incoming chat for the duration even though tail notifications keep arriving. Rely on the persistent tail to surface in-game signals, have the running task `bot.chat` periodic progress, and only check `TaskOutput` (non-blocking) when you specifically need the return value.

## Mineflayer + pathfinder

You drive the bot directly via mineflayer's API — no skill wrappers, no action DSL. Reference docs:

- mineflayer API: https://prismarinejs.github.io/mineflayer/#/api
- pathfinder: https://github.com/PrismarineJS/mineflayer-pathfinder#readme

Common things in eval scope:
- `bot.chat(text)` / `bot.whisper(user, text)`
- `bot.entity.position`, `bot.health`, `bot.food`, `bot.experience`
- `bot.inventory.items()`, `bot.heldItem`
- `bot.findBlock({ matching, maxDistance })`, `bot.findBlocks(...)`
- `bot.dig(block)`, `bot.placeBlock(block, faceVec)`
- `await bot.pathfinder.goto(new pathfinder.goals.GoalNear(x, y, z, 1))`
- `bot.entities` — map of nearby entities

## Persistent files

- `.runtime/skills/` — your own private JS snippets. Load with `eval`'s `require` (absolute path) or `Read` + inline. Edit, refactor, delete freely as you build experience.
- `.runtime/notes/` — your scratch space. World details ("diamonds at -120 12 340"), what worked, what failed, todos. Read at session start when relevant; write whenever you learn something durable.

Both dirs survive across sessions but are gitignored, so they're per-machine. Build them up over time. Don't reinvent skills you already wrote — check `.runtime/skills/` first.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [domdomegg/claude-code-plays-minecraft](https://github.com/domdomegg/claude-code-plays-minecraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
