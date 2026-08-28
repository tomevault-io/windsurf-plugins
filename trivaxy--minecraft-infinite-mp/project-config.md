---
trigger: always_on
description: **This landed as a feature during the bug-fixing phase below, at the user's explicit call**,
---

# Working notes for agents

## The command system was replaced with Brigadier, and every command deleted

**This landed as a feature during the bug-fixing phase below, at the user's explicit call**,
and it is the one thing on this page that is not a bug fix. Read it before touching anything
that runs or tests a command.

The old framework was an unfinished leftover: one abstract `Command`, a static `Command[256]`
with a static cursor, `holder.command.split(" ")`, and arity dispatch (`args.length == 4/3/2`)
that made `/tp Alice 1 2` parse as coordinates. Tab completion read a second, hand-written
`suggestions()` table that the parser never consulted, so the two disagreed. Non-`CommandException`
throwables — commonly an out-of-bounds `args[n]` — were logged at WARNING and the sender was
told nothing at all.

It now runs on `com.mojang:brigadier:1.3.10`. See DEVELOPING.md for the shape and for how a
command is registered. What matters here:

- **The dispatcher ships nearly empty.** All 64 commands were deleted, not translated — the
  user's call, on the grounds that the set was bloat. `CoreCommands`/`ClientCommands`/`ServerCommands`
  are the three registration hooks. Commands come back one at a time, and **`/give` is the first
  one back** (`commands/core/GiveCommand.java`, op-only, both sides). It takes the two forms the
  user asked for — `/give <item id>` and `/give <player> <item id>` — and no count argument, so
  it is *not* a drop-in for the twelve dead smoke suites below, which all called `/give <id> <count>`.
  It hands over one full stack. Read the DEVELOPING.md note on sibling argument branches before
  adding a second command shaped like it: registration order decides which form wins, and a
  branch that throws loses its error message to whichever sibling parsed.
- **Twelve smoke suites are dead as a result**, and this is known and accepted. The rig drove
  the server with `setblock` (27 uses), `teleport` (15), `weather` (8), `give` (8), `clearinv`
  (8), `gamerule` (6), `effect` (5), `time`/`summon`/`count` (3 each) and `kill`. `:server:smokeTest`
  cannot pass until this is dealt with. **The agreed follow-up is to give `Bot` direct
  server-side helpers** (`setBlock`, `giveItem`, …) so the rig stops depending on the command
  surface at all, rather than to bring those twelve commands back to serve it.
- **`/stop` is back** (`net.minecraft.server.commands.StopCommand`, op-only, dedicated server), initiating a graceful server shutdown.
- **`/kick <player>` is back** (`net.minecraft.server.commands.KickCommand`, op-only, dedicated server).
  It lives in `ServerCommands` rather than `CoreCommands` because `ConfigManagerClient.kickPlayer`
  is an empty method — registered in core it would be a command that silently does nothing in
  singleplayer. It answers the sender *before* it kicks, so kicking yourself still confirms.
  Covered by `KickSmokeTest` (needs `kickbot` in `run/server/ops.txt`).
- **`/ban <player> [reason]` and `/unban <player>` are back** (`BanCommand`, `UnbanCommand`,
  op-only, dedicated server). **Nothing new was written to make them work** — `BanList`,
  `BanEntry`, `banned-players.txt` and the login refusal in `ServerConfigurationManager.login`
  were all still there, unreferenced by any command since the purge; these two are a front end
  onto `ConfigManager.banPlayer`/`pardonPlayer`. `/ban` takes a name, not an online player, so
  an absent player can be banned; if they *are* online it kicks them with the same wording the
  login refusal uses. The reason is a greedy trailing string and is optional; omitted, `BanEntry`
  supplies its own default ("Banned by an operator."). `BanEntry` also carries a `banEndDate`
  that nothing sets — **temporary bans are already modelled and already expire** (`removeExpiredBans`
  runs on every `isBanned`), so a `/tempban` would need a date parser and nothing else.
  `/unban` reads `getBannedPlayers().isBanned` (a `ServerConfigurationManager` cast —
  `ConfigManager` does not expose it) so a typo comes back as "That player is not banned."
  rather than silently doing nothing. Covered by `BanSmokeTest` (needs `banbot` in
  `run/server/ops.txt`); it always pardons what it banned, and proves the pardon by logging
  back in. **Together, /kick /ban /unban moved `HelpSmokeTest`'s hardcoded op command list from
  three entries to six.**
- **`/tp <x> <y> <z>` is back** (`commands/core/TpCommand.java`, op-only, both sides — the
  client implements `teleportPlayerTo` for real, so unlike `/kick` it belongs in `CoreCommands`).
  **Nothing new was written for the coordinate parser either**: `Vec3Argument`, `WorldCoordinate`
  and `WorldCoordinates` already parsed `10 20 30`, `~ ~5 ~` and `~-2.5 ~ ~1.25`, and had no
  caller outside `CommandFrameworkSmokeTest`'s scratch tree. It teleports the sender, and the
  sender of a command block's line is the nearest player within 16 blocks — so `/tp ~ ~ ~` in a
  command block yanks that player onto the block (`~` is the block, the mover is the player),
  and with nobody in range it is "That command has to be run by a player." That source carries a
  null `ConfigManager` (nothing in `:core` can hand it one), so `/tp` moves the entity directly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Trivaxy/Minecraft-Infinite-MP](https://github.com/Trivaxy/Minecraft-Infinite-MP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
