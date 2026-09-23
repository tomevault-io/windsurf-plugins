---
trigger: always_on
description: A World of Warcraft addon (folder + `.toc` name: **`Aegis_Exchange`**) — a clean
---

# CLAUDE.md — Aegis: Exchange

A World of Warcraft addon (folder + `.toc` name: **`Aegis_Exchange`**) — a clean
auction house helper for **Turtle WoW 1.18.1**, which runs the **ORIGINAL WoW
1.12 (vanilla) client on Lua 5.0**.

> This is **NOT** WoW Classic and **NOT** retail. Do **not** use any API newer
> than patch **1.12**. When in doubt, assume the API does not exist.

**Planned work lives in [`ROADMAP.md`](ROADMAP.md)**, phased and dependency
ordered, including the integration contract for the planned **Aegis: Courier**
companion addon (separate repo). Check it before starting any large feature
so the phase ordering and settled design decisions aren't re-litigated or
built out of order.

---

## HARD RULES — never violate these

These are not style preferences. Breaking any of them produces a runtime error
or silent breakage on the 1.12 / Lua 5.0 client.

### Language (Lua 5.0)

1. **Lua 5.0 only.** **NO** `string.match`, **NO** `string.gmatch`, **NO**
   `:match()`. Use **`string.find`** (with captures) and **`string.gfind`**.
   - `string.gfind` is the 5.0 name for what later Lua calls `string.gmatch`.
2. **NO `#` length operator.** Use **`table.getn(t)`**. **NO `table.setn`.**
3. **NO `%` modulo operator.** Use **`math.mod(a, b)`**.
   - Lua 5.0 also has no integer division — combine `math.floor` with
     `math.mod`.
4. **Varargs use the `arg` table and `arg.n`** — not `...` expansion helpers
   from later versions. (`select()` does not exist.)
5. String library note: `string.gsub`, `string.find`, `string.gfind`,
   `string.format`, `string.sub`, `string.lower`/`upper` are fine. The banned
   ones are strictly the `match`/`gmatch` family.

### Events

6. **Event handlers read the GLOBALS `event`, `arg1`, `arg2`, …** — **NOT**
   `function(self, event, ...)`. On this client the OnEvent script receives no
   arguments; the client sets `this`, `event`, and `arg1..argN` as globals.
   - Central dispatch lives in `core/init.lua`. Register with
     `AegisExchange.RegisterEvent(evt, fn)`; the dispatcher reads the globals
     and forwards them.

### Hooking

7. **NO `hooksecurefunc` and NO secure hooks.** Hook by **saving the original
   function and replacing it**, then call the saved original from your
   replacement. (Secure-hook infrastructure does not exist in 1.12.)
   See `ui/tooltip.lua` for the canonical pattern.

### Auction House API (1.12)

8. **`GetAuctionItemInfo("list", i)`** returns **ONLY** these values, in order:
   ```
   name, texture, count, quality, canUse, level,
   minBid, minIncrement, buyoutPrice, bidAmount, highBidder, owner
   ```
   Nothing else. **`owner` may be `nil`** until the name resolves — re-read the
   page or handle nil gracefully.
9. **`QueryAuctionItems` takes 9 args:**
   ```
   QueryAuctionItems(name, minLevel, maxLevel, invTypeIndex,
                     classIndex, subclassIndex, page, isUsable, qualityIndex)
   ```
   - **`page` is 0-indexed.**
   - **There is NO working `getAll` on 1.12.** Do not attempt a bulk pull.
   - Pass **strings** for `name` / `minLevel` / `maxLevel` — **`""` when
     unused, never nil**. The stock browse UI sends `GetText()` results
     (always strings) and Auctionator does the same; servers may silently
     ignore a query with nils in those slots. The index/flag args
     (`invType`, `class`, `subclass`, `isUsable`, `quality`) stay nil for
     "no filter".
10. **Throttle every query.** Poll **`CanSendAuctionQuery()`** before **every**
    query — that gate is the authority, never a wall-clock timer alone. Wait
    for the **`AUCTION_ITEM_LIST_UPDATE`** event before reading a page.
    - The client keeps the gate shut ~5s after each query, which is where the
      old "leave ~4 seconds between pages" rule of thumb came from. We now
      apply only a small floor (`scan.FAST_DELAY`) and let the gate do the
      throttling, because the **AuctionQueryThrottle** DLL
      (<https://github.com/brues-code/AuctionQueryThrottle>) clears that timer
      as soon as the reply lands. It is a DLL, **not an addon** — there is
      nothing to `IsAddOnLoaded()`, so the gate itself is the detector: it
      opens fast with the DLL and stays shut ~5s without it.
    - `scan.PageDelay()` returns the floor for the current pacing setting;
      "safe" restores the fixed 4s for clients that report the gate unreliably.
      **Never** send a query without checking the gate, whatever the floor.
11. **Page size is 50.**
12. **Hiding `AuctionFrame` ENDS the AH session.** `AuctionFrame`'s XML
    `<OnHide>` runs **`CloseAuctionHouse()`**, so **any** `AuctionFrame:Hide()`
    / `HideUIPanel(AuctionFrame)` closes the server session and every following
    `QueryAuctionItems` becomes a silent no-op (a scan spins forever on
    "Requesting first page…"). Our standalone window replaces the Blizzard AH,
    so it must hide `AuctionFrame` **without** letting that `<OnHide>` body run:
    save-and-replace its `OnHide`, and while *we* are the one hiding it, skip
    the default body so the session survives. See `ui.HideBlizzardAH` /
    `ui.HookAuctionFrame` in `ui/frame.lua`.
    - **A SECOND close path lives in `AuctionFrame_Show()`** (the client's
      AUCTION_HOUSE_SHOW handler, verbatim from the Turtle UI source):
      ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Torchlite-bit/Aegis_Exchange](https://github.com/Torchlite-bit/Aegis_Exchange) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
