---
trigger: always_on
description: A learning platform and building tool for real-time graphics with SDL's GPU API.
---

# forge-gpu

A learning platform and building tool for real-time graphics with SDL's GPU API.

**Dual purpose:**

1. **Learn** — Guided lessons teaching GPU programming, math, and game techniques
2. **Forge** — Skills and libraries enabling humans + AI to build real projects

Repository: https://github.com/RosyGameStudio/forge-gpu
License: zlib (matching SDL)

## Tone

Many users are learning graphics or game programming for the first time. When
answering questions or writing code:

- Be encouraging and patient — there are no dumb questions about GPU APIs or math
- Explain *why*, not just *what* — connect each concept to the bigger picture
- Use plain language before jargon; when jargon is unavoidable, define it
- Suggest alternatives gently rather than just correcting
- Reference specific lessons when relevant
- When using the math library, link to both the library docs AND the math lesson

### Lesson writing tone

Lessons teach real techniques backed by math and engineering — treat every
concept with the respect it deserves.

- **Banned words:** *trick*, *hack*, *magic*, *clever*, *neat* — these cheapen
  the material. Use instead: *technique*, *method*, *approach*, *insight*,
  *key idea*, *shortcut*, *property*, *observation*.
- **Be direct and precise.** State what a technique does and why it works.
  Avoid hedging phrases ("it turns out that", "there's a neat way to").
- **Credit named techniques** (Blinn-Phong, Gram-Schmidt) — they carry weight
  and help readers find more resources.
- **Explain reasoning, not magic.** "We use the transpose because rotation
  matrices are orthonormal" beats "there's a neat trick with the transpose."

### Documentation and public-facing tone

READMEs, descriptions, and any text a visitor reads first. The core audience
is two groups: beginners new to graphics/game dev, and experienced graphics
programmers who evaluate projects critically. Expert endorsement is what drives
adoption — if credible sources recommend forge-gpu, beginners follow.

**Say what it is. Don't sell it.**

- **No rhetorical flourishes.** "The same license as SDL" not "the same license
  as SDL itself." Every unnecessary word that exists to make something sound
  more impressive undermines credibility.
- **No AI-generated hype.** Words like *powerful*, *unlock*, *robust*,
  *grounded*, *revolutionary* are instant dismissal for the expert audience.
  They pattern-match to LLM output.
- **No selling against others.** "Most tutorials only do X" is insecure.
  Present the project's own value and let people compare.
- **No performing.** "Designed for exploration with an AI assistant" is
  performing. "The lessons are structured so Claude Code can navigate them" is
  informing. The difference is whether the sentence creates an impression or
  communicates a fact.
- **Earned confidence over enthusiasm.** The tone of someone who built
  something substantial and can describe it plainly — not someone trying to
  convince you it's substantial.
- **Stable facts.** If a number will go stale next week (lesson counts), don't
  feature it prominently. Describe scope through arcs and breadth instead.
- **Trust the reader.** Short is fine. Sparse is fine. Over-explaining reads
  as selling. If the information is there, people will find it.

## Code conventions

- **Language:** C99, matching SDL's own style conventions
- **Naming:** Types use `PrefixPascalCase` (e.g. `ForgeCapture`,
  `ForgePhysicsBody`), functions use `prefix_snake_case` (e.g.
  `forge_capture_init`, `forge_physics_integrate`), local variables use
  `lowercase_snake`. The prefix is the module name (`forge_capture_`,
  `forge_physics_`, etc.) — NOT `SDL_`, which is reserved for SDL's own
  symbols. The math library uses short lowercase names (`vec3`, `mat4`,
  `quat`) following GLSL/HLSL convention
- **Constants:** No magic numbers — #define or enum everything
- **Comments:** Explain *why* and *purpose* (every uniform, pipeline state, etc.)
- **Errors:** Handle every SDL GPU call with descriptive messages. **Every SDL
  function that returns `bool` must be checked** — log the function name and
  `SDL_GetError()` on failure, then clean up resources and early-return. This
  includes `SDL_SubmitGPUCommandBuffer`, `SDL_SetGPUSwapchainParameters`,
  `SDL_Init`, and others. Never ignore a bool return value.
- **Line endings:** Always use Unix-style (LF) line endings — never CRLF.
  The repository enforces this via `.gitattributes`.
- **Readability:** This code is meant to be learned from — clarity over cleverness
- **glTF assets:** Load entire models via `forge_gltf_load()` — never extract
  individual textures or meshes à la carte. Copy the complete model (`.gltf`,
  `.bin`, textures) into the lesson's `assets/` directory.
- **Builds:** Always run build commands via a Task agent with `model: "haiku"`,
  never directly from the main agent.

### No sentinel patterns on stack-allocated structs

**Never use a "magic number" sentinel field to detect whether a
stack-allocated struct has been initialized.** In Release builds, stack
memory is not zero-initialized — it contains leftover data from previous
function calls. If the sentinel value happens to appear at the right

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nebulavenus/forge-gpu](https://github.com/Nebulavenus/forge-gpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
