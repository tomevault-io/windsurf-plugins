---
trigger: always_on
description: CURRENT ARCHITECTURE:
---

CURRENT ARCHITECTURE:
Use the recipe-cartridge split.

The engine owns durable functionality:

* input actions and keyboard/gamepad polling
* gamepad Learn UI and saved mappings
* pause/status-bar menu
* audio context / RetroCharm SFX path
* physics, collision, camera, update/render loop
* entity behavior: enemies, powerups, blocks, portal, player states
* recipe dispatch
* reusable drawing primitives and prefabs
* HUD behavior, overlays, panels, attributions UI
* external cartridge loading/normalization/validation

The cartridge owns reproducible/changeable world content:

* title, HUD label text, messages
* world dimensions and player spawn
* platform layout
* floating blocks and block contents
* coins, stars, enemies, signs, portal
* theme data
* drawing recipes
* optional audio overrides
* optional resource/attribution records
* optional decorative "resourceScenery"

The boundary is practical, not purist. The engine can include rich drawing functions/prefabs such as mushroom mascot, blob, portal, coin, star, grass platform, rounded block, HUD panel, parallax sky, glow, particles, oval, capsule, roundedRect, swirl, trees, mushrooms, pipes, arches, etc. The cartridge feeds custom data into those functions. Avoid putting custom "drawPortal()"-style functions in normal cartridges unless intentionally making a trusted custom/hack cartridge.

EXTERNAL CARTRIDGES:
External cartridges use JSON and should normally be named:

something.llmcart.txt

Android file pickers may only expose ".txt" reliably, so ".llmcart.txt" is the current working convention.

Expected cart format:
format: "llmcart-toybox-recipe"
cartVersion: 2

PLACEMENT / CLEANUP CONTRACT:
Generated or draft cartridges should prefer the normal, managed placement path. The loader runs a conservative one-time cleanup pass on external cartridges, after JSON parse and before gameplay reset. Its job is to prevent recurring draft mistakes, not to redesign a finished level.

* Surface props: resourceScenery pipes, trees, shrubs, mushrooms, and arrow signs are treated as surface-bound unless they declare anchor:"raw". The normalizer finds the closest valid walking surface near the prop and places the prop base cleanly on that surface.
* Pipes marked solid:true rebuild their collision from the corrected pipe base.
* Surface props that cannot be placed on a nearby surface are dropped rather than left floating over a chasm. Use anchor:"raw" for intentional weird/floating decorative placement.
* Floating blocks are binary: either their bottom is snapped to the walking surface as a grounded/static block, or their underside is high enough for the big hero plus padding to walk underneath. The forbidden middle zone is normalized away to prevent small-hero-only crawl spaces and big-hero teleport/shove bugs.
* Final hand-authored carts can opt out of all placement cleanup with placement:{autoCleanup:false}; individual objects can opt out with anchor:"raw".

The loader should:

* load from the pause menu
* parse JSON
* validate/normalize expected level fields
* reset gameplay state after loading
* replace world arrays from the external cart
* inherit built-in recipes/audio/resources unless overridden
* avoid leaking demo-only decorative "resourceScenery" into loaded carts unless the loaded cart provides its own
* keep the engine durable; cartridges should not need to understand the whole engine

CURRENT INPUT STATE:
Keyboard:

* arrows / WASD movement
* Space / Up / W jump
* Shift / Z / X run
* P / Escape / Enter pause
* R restart

Default gamepad mapping:

* Left: Button 14
* Right: Button 15
* Up: Button 12
* Down: Button 13
* Run: Button 0
* Jump: Button 1
* Pause/Start: Button 9

Gamepad Learn lives in the pause/status-bar menu. The visible always-on gamepad icon was removed. Tapping/clicking the top status bar opens pause/settings.

CURRENT AUDIO STATE:
RetroCharm SFX are integrated and working. The engine has one shared audio path intended to support future theme music/MIDI work later.

Current built-in SFX hooks include:

* jump
* coin
* star
* bonk
* mushroom emerge
* powerup collect
* enemy stomp
* hurt/shrink/life loss
* level clear

The audio system is intentionally ready for future music, but full MIDI parser/drop-folder/song scheduling is not part of the current baseline.

CURRENT GAMEPLAY STATE:
Current built-in cartridge is the Physics Test Plains / Jump Test style level. It is a known-good gameplay baseline with:

* working run/jump/fall tuning
* standing and running jumps that feel fair
* mushroom powerup block
* big/small player loop
* enemies, stomps, shrink/hurt/lives
* coins and stars
* bonk blocks and messages
* moving/bobbing floating blocks
* portal win condition
* level clear overlay
* signs and speech bubbles
* counters/HUD updating correctly without overlap

Current physics tuning has been validated by play feel. Avoid casual changes to jump/fall/run constants unless deliberately doing a physics pass.

CURRENT VISUAL STATE:
The current hero is a derivative mushroom-cap mascot. It is much stronger than the previous round/helmet/jellybean-headed mascot. Preserve the basic silhouette unless intentionally redesigning.

Grass/dirt rendering lesson:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thesepeoplearenotyourfriends/super_llmario](https://github.com/thesepeoplearenotyourfriends/super_llmario) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
