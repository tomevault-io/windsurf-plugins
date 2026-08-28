---
trigger: always_on
description: A 128×128, eight-colour fantasy console that lives in an Omarchy bar widget.
---

# micromachee

A 128×128, eight-colour fantasy console that lives in an Omarchy bar widget.
One Lua file per game.

## How it is put together

```
manifest.json   what Omarchy reads
Panel.qml       the bar button and the panel — drawing only
Service.qml     runs the helper, turns its output into properties
helper/         Rust: everything that can actually fail
carts/          the games
themes/         palettes + the rules a palette must keep
```

**The split is the point.** QML cannot be unit-tested without a compositor, so
anything written there is only ever verified by looking at it. Anything in
`helper/` is reachable by `cargo test` on any machine. Logic goes in the helper;
the QML layer stays dumb enough to be obviously correct.

Two conventions the QML depends on:

- `status` prints **one line of JSON** and exits 0. It is polled on a timer, so
  it must be cheap and must never block.
- Every other command prints a **human sentence** to stderr and exits non-zero
  when it fails. The panel shows that last line verbatim — write it for the
  person reading the bar, not for a log.

```bash
cargo test  --manifest-path helper/Cargo.toml
./install.sh                      # after changing the helper
```

---

# Writing a cart

This is the part most sessions are here for. A cart is **one Lua file, at most
24K**. It needs `_draw()`. `_init()`, `_update()` and `_cover()` are optional.
30 frames a second.

```lua
-- title: Dodge
-- author: you
-- about: one line, optional

local x

function _init()  x = 64 end
function _update()
  if btn(0) then x = x - 2 end
  if btn(1) then x = x + 2 end
  x = mid(0, x, 127)
end
function _draw()
  cls(0)
  print("SCORE 0", 2, 2, 7)
  rect(x - 3, 118, 7, 4, 6)
end
```

The `-- title:` / `-- author:` / `-- about:` comments are optional metadata, not
a required header. There is no magic line. If you write valid Lua under 24K with
a `_draw`, it is a cart.

## The whole API

```
cls(c)  pset(x,y,c)  pget(x,y)
rect(x,y,w,h,c)  rectb(x,y,w,h,c)  line(x0,y0,x1,y1,c)
circ(x,y,r,c)  circb(x,y,r,c)  print(text,x,y,c,scale)
btn(i)  btnp(i)  t()  rnd(n)  flr(n)  mid(lo,v,hi)  score(n)
save(key,value)  load(key)  now()  lose()  win()
```

That is all of it. There is no sprite sheet and no sound. Lua's `math.*`,
`table.*` and `string.*` are available.

**A cart gets `math`, `string` and `table`, and nothing else.** `io`, `os`,
`package`, `require`, `dofile` and `loadfile` are all absent, so a cart cannot
open a file, run a program, or reach anything outside the console. This was not
always true: the first version loaded the whole standard library, and a test
cart wrote a file to `/tmp` to prove it. Since `sync` pulls carts off a CDN,
that was a hole rather than a footnote.

It is still not a security boundary in the strong sense — it is a Lua
interpreter in your process, and the per-frame instruction budget and memory
ceiling exist to stop a cart hanging your bar rather than to contain a
determined attacker. Treat a cart from a stranger the way you would any script
they sent you: read it first. It is one file and at most 24K, which is the
point.

- `btn(i)` is held, `btnp(i)` is newly pressed this frame.
  **0** left · **1** right · **2** up · **3** down · **4** O · **5** X
- `t()` is seconds since the cart started. `rnd(n)` is a float in `[0, n)`.
- `mid(lo, v, hi)` clamps — the usual way to keep a player on screen.
- `score(n)` is fire-and-forget. The console owns high scores per cart; a cart
  cannot read or lower them. Call it when the score changes.
- `pget(x,y)` reads the framebuffer back, so you can collide against what you
  drew last frame instead of keeping a parallel model. Very effective for cave,
  tunnel and maze games.
- **`lose()`** at the moment the player fails — the same line where you already
  set `alive = false`. It changes nothing in normal play, and it is the only
  thing that lets **Mega Micromachee** tell whether you survived your few
  seconds of your game. A cart that never calls it always survives its round.
  `win()` is the mirror, for a game that can be finished.
- `save(k,v)` / `load(k)` persist across runs — numbers, strings and booleans
  only. `now()` is wall-clock seconds since 1970, where `t()` is seconds since
  this run began. Together they make time pass while the console is closed:
  store *when* something started and compare, never a countdown, because
  nothing counts down while the widget is shut. `farm.lua` is the worked
  example.
- `flr` and `mid` both return an **integer** when given integers, so
  `"p" .. mid(0, x, 3)` is `"p2"` and not `"p2.0"`. That mattered the day a
  game started using a clamped coordinate as a save key.
- `print`'s fifth argument is an optional **scale**, defaulting to 1: every
  pixel of the glyph becomes a block that many wide. It is how you get a title
  worth looking at out of a 3x5 font. A scaled line is `#text * 4 * scale`
  pixels wide, so centring is `(128 - #text * 4 * scale) / 2`.

## Give it a cover

`_cover()` draws the picture the shelf shows, and the one that comes up full
size before the game starts. It runs once, after `_init()`, on the same 128x128
screen with the same eight colours:

```lua
function _cover()
  cls(0)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pixygon/micromachee](https://github.com/Pixygon/micromachee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
