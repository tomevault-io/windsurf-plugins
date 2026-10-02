---
trigger: always_on
description: How the plugin works, where things live, and how to work on it. The README is
---

# claude-toons: notes for agents

How the plugin works, where things live, and how to work on it. The README is
for users: install and cost. Keep technical detail here.

## How it works

- **Watching.** A `tool.call` hook logs each tool Claude runs: command, file,
  pattern, and whether it failed. `prompt.submit` and `turn.complete` mark the
  turn's edges, and the spinner's mode tells when Claude turns to thinking or
  to writing its reply.
- **Scene sources.** The "Scenes" setting (config key `library`): "ready-made
  only" never asks the director and spends nothing; "mix", the default, deals
  ready-made scenes and asks the director for news; "fresh only" asks the
  director for everything, at the pace setting. Old saved values map over:
  on to mix, off to fresh only.
- **Ready-made scenes.** Used in "mix" and "ready-made only". `hooks/library.ts` reads the log for
  what Claude is doing (thinking, searching, reading, editing, testing,
  building, running, git, web, agents, writing) and the file or command it
  last touched. Routine news deals a stock scene from `hooks/scenes.ts` for
  that phase, with `{what}` in it filled by the file or command. A scene
  keeps the strip for at least 20 seconds (45 for one from the director),
  then gives way when the work turns to a new phase, or after 60 seconds in
  any case; a scene whose code broke goes at once (`isStockDue`). Thinking
  between tool calls doesn't count as a new phase. The pace setting doesn't
  apply to stock scenes. Each phase has a shuffled deck, so no scene repeats until
  the rest of its deck has played. Only scenes in the styles the setting
  allows are dealt; a scene's style is what its code really calls, not what
  it was asked for.
- **The director.** A model asked for a fresh scene. In "mix" it is asked only for news worth it (the task, a failed or denied tool,
  a scene whose code broke, or a phase with no stock in the allowed styles),
  and at most once a minute. In "fresh only" it is asked whenever there
  is news, at most once per the pace setting. The log goes to the director in
  one conversation per session, so each request reads the earlier ones from
  the prompt cache. Only the last two scenes stay whole in the thread; older
  ones collapse to their one-line concept, a batch of six at a time so the
  cached prefix is rebuilt rarely. The thread is cut to 30 messages at 60.
  The reply is a scene as JSON (structured outputs).
- **The prompt.** `systemFor()` in `hooks/narrator.ts` builds the director's
  prompt from only the drawing calls the allowed styles need: no 3D calls or
  3D example for "no 3D", no pixel calls for text art. A request names a style
  only when more than one is allowed. Keep the prompt about intent; don't add
  menus of jokes or formats, which make every scene the same.
- **Quiet stretches.** While nothing new happens, the scene keeps animating
  on its own and nothing is requested. After 90 seconds on one scene, a
  "still running" line asks for the scene's next beat, or deals a new stock
  scene when ready-made scenes are on.
- **Scenes.** A faint backdrop effect, particle swarms, actors (frames of
  ASCII art moved by math expressions of time, like
  `x = "mod(t*8, w+20) - 20"`), and optionally a program.
- **Code.** A scene can carry a program in a JavaScript-like language,
  interpreted by `hooks/lang.ts`: closures, loops, arrays and objects,
  destructuring, template literals, `switch`, Math, and the common array and
  string methods. The top level runs once; `frame(t, dt)` redraws every frame
  and state persists between frames. It draws with `put`, `text`, `sprite`,
  `fill`, `line`, `circle` and `disc` in any colors, `clawd()` for the mascot
  and `say()` for a speech bubble.
- **Pixels.** Cells hold two pixels each (upper and lower half blocks), and
  code draws on that grid with `pixel()` and `pixels()`. Clawd is drawn on the
  same grid, with poses, a walk cycle, eye states, claw poses, any color and
  up to 4x scale; `clawd()` returns anchors (head, claw tips, feet, eyes) for
  hats and props.
- **3D.** `hooks/render3d.ts`. The world is sampled at 2x4 points per cell,
  lit with smoothly interpolated normals, and each cell becomes the
  quarter-block glyph that best splits its samples into two colors. A dark
  rim marks depth jumps, and fog reads as depth. A mesh drawn `ascii` uses
  glyphs by brightness. `clawd3d()` stands Clawd at a world point as a lit
  solid with flat pixel-art eyes, both on one row, and `project()` maps a
  world point to the strip.
- **Sandbox.** Plugins have no `eval`, and scene code comes from a model
  reading the user's repo, so it runs in the interpreter alone and reaches
  nothing but its own values and the drawing calls. Every step burns fuel
  (1M for setup, 150k per frame) and a frame may run at most 80ms, so a
  runaway loop stops the scene, not the terminal. An error in a director
  scene is reported back so the director can fix it; a broken stock scene is
  replaced from the stock.
- **Drawing.** A `ui.render` hook on `Spinner` keeps the engine's spinner
  line and adds a `Raster` under it, repainted at about 20 fps with
  `$.ui.blit`. Empty cells show the terminal's background. A new scene

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [achimala/claude-toons](https://github.com/achimala/claude-toons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
