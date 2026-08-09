---
trigger: always_on
description: You're here to play the mmo game through the progressive development of botting scripts, starting small then adapting to your desires and ideas.
---

# RS-Agent Bot Guide

You're here to play the mmo game through the progressive development of botting scripts, starting small then adapting to your desires and ideas.
It is strongly recommended to get started and make the first step towards your goals, then researching and learning as you go.

## First Time Setup

**Create a new bot using the setup script:**

Ask the user for a bot name (max 12 chars, alphanumeric). If they skip, use the command without a username to auto-generate a random 9-character name.

```bash
# With custom username
bun bots/create-bot.ts {username}

# Auto-generate random username
bun bots/create-bot.ts

# Use local server (sets SERVER=localhost in bot.env)
bun bots/create-bot.ts {username} --local

# Use a custom server
bun bots/create-bot.ts {username} --server=myserver.example.com
```

This automatically creates:

- `bots/{username}/bot.env` - Credentials with auto-generated password
- `bots/{username}/lab_log.md` - Session notes template
- `bots/{username}/script.ts` - Ready-to-run starter script

### Quick Start

1. Install dependencies: `bun install` (from project root)
2. Open project in your AI coding tool — the MCP server will be available automatically
3. Control your bot with suggestions.

### Tools

| Tool                           | Description                                    |
| ------------------------------ | ---------------------------------------------- |
| `execute_code(bot_name, code)` | Run code on a bot. Auto-connects on first use. |
| `list_bots()`                  | List connected bots                            |
| `disconnect_bot(name)`         | Disconnect a bot                               |

### Example

```typescript
// Just execute - auto-connects on first use
execute_code({
  bot_name: "mybot",
  code: `
    const state = sdk.getState();
    console.log('Position:', state.player.worldX, state.player.worldZ);

    // Chop trees for 1 minute
    const endTime = Date.now() + 60_000;
    while (Date.now() < endTime) {
      const tree = sdk.findNearbyLoc(/^tree$/i);
      if (tree) await bot.chopTree(tree);
    }

    return sdk.getInventory();
  `,
});
```

See `mcp/README.md` for detailed API reference.

Code runs in an async context with `bot` (BotActions) and `sdk` (BotSDK) available as globals.

## Reporting SDK Bugs

When the SDK has a bug or rough edge, first find a workaround, then file a report. One command, no auth, no permission needed:

```bash
bun sdk/bug-report.ts "incorrect results from bot.foo(), had to use raw sdk.sendFoo() instead for xyz reason."
```

Important note: The game itself is extremely well tested and complete, it's not buggy. If you can't figure out how to do something, don't blame the game, blame your assumptions and keep investigating. File a bug report _after_ you figure out
what's going on, don't just assume the game is broken because you got confused, and give up. The SDK is a thin layer on top of the game, and the game is the source of truth.

After filing the bug, keep working on your goal!

## Session Workflow

This is a **persistent character** - you don't restart fresh each time. The workflow is:

### 1. Check World State First

Before writing any script, check where the bot is and what it has:

```bash
bun sdk/cli.ts {username}
```

This shows: position, inventory, skills, nearby NPCs/objects, and more.

**Exception**: Skip this if you just created the character and know it's at spawn.

**Tutorial Check**: If the character is in the tutorial area, call `await bot.skipTutorial()` before running any other scripts. The tutorial blocks normal gameplay.

### 2. Write Your Script

Edit `bots/{username}/script_name.ts` with your goal. Keep scripts focused on one task. you may write multiple scripts for different tasks and switch between them.

### 3. Run the Script

```bash
bun bots/{username}/script_name.ts
```

### 4. Observe and Iterate

Watch the output. After the script finishes (or fails), check state again:

```bash
bun sdk/cli.ts {username}
```

Record observations in `lab_log.md`, then improve the script.

### Chatting Without Taking Control

To talk in-game (or read chat) without disturbing whatever script currently
owns the bot, use the chat CLI — it connects in `observe` mode, which can send
chat (and nothing else) and never pre-empts (or gets pre-empted by) a
controller:

```bash
bun sdk/chat.ts {username} "meet me at the bank"   # send
bun sdk/chat.ts {username}                          # recent chat
bun sdk/chat.ts {username} --watch                  # tail live
```

Public chat is capped at 400 chars per message; `sdk.say()` auto-splits longer
text into wire-safe chunks.

## Script Duration Guidelines

**Start short, extend as you gain confidence:**

| Duration      | Use When                                              |
| ------------- | ----------------------------------------------------- |
| **10s**       | New script, single actions, untested logic, debugging |
| **30s-1 min** | Validated approach, building confidence               |
| **5+ min**    | Proven strategy, grinding runs. USE SPARINGLY         |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaxBittker/rs-sdk](https://github.com/MaxBittker/rs-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
