---
trigger: always_on
description: - Keep the game local. Do not create, connect, publish, or deploy a Site unless the user explicitly requests it.
---

# Working on Evergrow

## User preferences

- Keep the game local. Do not create, connect, publish, or deploy a Site unless the user explicitly requests it.
- Use the Codex in-app browser for previews, reusing the existing local game tab.
- The user does gameplay testing and provides feedback on combat feel, movement, balance, and visuals. Do not drive gameplay, launch another browser, or run automated browser playtests unless explicitly asked.
- The user requested town/city layout captures on 2026-09-05. Static scene staging and captures in the in-app browser are authorized for that request; keep their playable session separate.
- Code-level tests, type checking, and production builds are appropriate. Existing optional browser regression tests require the user's explicit request before running.
- Keep the local development server available while the user tests. Avoid unnecessary reloads during their play session.
- Commit coherent changes as checkpoints and push each checkpoint to origin, as requested by the user. Keep the existing remote; do not add or change remotes unless requested.
- This is an unreleased prototype for the user’s own testing. Prefer the cleanest current design over backward compatibility. When replacing a system, update its callers/tests and remove obsolete implementations, exports, adapters, and legacy-only tests. Do not keep old features or build compatibility layers for hypothetical consumers. Old save formats may be invalidated when a design change requires it; migrations are not required at this stage. Mention any resulting test-progress reset. Git checkpoints provide the history.

## Current scope

The local slice now includes seven smoothly blended climates: Deadwood, Verdant Forest, The Mire, Frostpine Reach, Emberfall, Amberwood, and Hollow Highlands. A seeded two-dimensional climate field drives terrain, 23 procedural prop families, ambient light, encounter/camp composition, and gear-profile biases. Procedural settlements have enterable buildings, and exploration maps show the visited world at multiple scales. Continue to prioritize the procedural asset engine, smooth transitions, readable navigation, and satisfying combat. The first character foundation now includes equipment, inventory, attributes, a large skill atlas, seventeen unlockable active skills, one-/two-handed melee gear, bows, elemental staves, shields, dual wield, and enemy gear drops. Six enemy archetypes now patrol, sense line of sight, commit to telegraphed attacks, and return to a home tether. Seeded camps retain member health/deaths for the run; four additional wilderness landmark types appear on the explored map. Eight local character slots now persist progression, gear, resources, world position, ground loot and camp casualties; each character has a separate explored chart. Town blacksmiths, jewelers and enchanters now provide trading, buyback, +10 enhancement, rarity upgrades, affix rerolls and geographic releveling. Cloud saves and deeper item/skill content remain future work. Read `docs/character-systems.md` and `docs/weapons-and-skills.md` before extending these systems. `docs/README.md` indexes current guides, proposals and historical records; `docs/system-status.md` is the current implementation summary. Update affected guides when changing rules; keep past checkpoint counts explicitly historical. Implemented NPC/vendor rules and initial tuning live in `docs/npcs-and-vendors.md`. Save payload v2 includes explicit item recipes and commerce state; older slots are preserved but need new characters.

The Astral skill row contains basic attack (LMB) and exactly five assignable skills (RMB, 1–4), empty on a new run. Potion (Q) and dodge (Space) occupy separate utility shortcuts beside the menu rail. Unassigned controls do nothing. Seventeen major tree nodes unlock actual skills; the player assigns unlocked skills to the five slots. LMB supplies the equipped weapon’s innate melee swing, arrow, or elemental bolt without mana or a skill unlock. There is no universal right-click cast. Fireball is an explicitly unlocked, assigned, staff-required skill. Incompatible gear disables activation without removing assignments. Wand weapon attacks remain future work.

Gold is a per-character wallet (`wallet.ts`), credited by physical coin piles (`gold.ts`) and item sales. All NPC debits/credits must use the same validated wallet API. Keep currency rolls independent from equipment RNG, persist wallet and ground coins atomically, and route gold/XP gains into one accumulating `rewards` notification in the shared feed. Keep item pickups individually named. `reward-feedback.ts` owns bounded presentation only; `/rewards.html` is a frozen, save-free review.

Starting mana regeneration is 1/second; equipment and passive bonuses add to it. Q is a dual potion restoring 42% maximum life and 40% maximum mana with one shared charge, usable if either resource is missing. Its two charges, 0.8-second base cooldown and one charge per eight kills live in `PLAYER_ABILITIES.potion`. Potion feedback carries actual life/mana amounts separately from healing-only skill events.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dimillian/Evergrow](https://github.com/Dimillian/Evergrow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
