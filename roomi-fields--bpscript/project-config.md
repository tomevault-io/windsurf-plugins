---
trigger: always_on
description: > ⚠️ **CONTEXTE BPx UNIQUEMENT (règle dure, Romain 2026-06-16).** L'AST produit par le parser est
---

## BPScript — Meta-sequencer for Temporal Structure Composition

> ⚠️ **CONTEXTE BPx UNIQUEMENT (règle dure, Romain 2026-06-16).** L'AST produit par le parser est
> **agnostique du moteur** et destiné à **BPx** — il ne doit contenir AUCUNE notion BP3 (`_xxx(N)`,
> `flavor:'bp3'`, catégorie « bp3 »…). La sortie **BP3** (`compileBPS().grammar`, ancienne fonction
> « BPScript → BP3 ») est **héritée : NE JAMAIS Y TOUCHER** sauf demande **claire et explicite**.
> Toute taxonomie d'AST se conçoit agnostique (ex. `target: transport|engine`, `timing: bang|durée`),
> jamais « bp3 vs bpx ». Cf. mémoire `feedback_bpx_only_jamais_bp3`.

3 reserved words, 24 symbols, 9 flag operators. Compiles to BP3 grammar format and runs via WASM.
Orchestrates SC, TidalCycles, Python, MIDI, DMX, etc. in a single file via backticks.

### Language summary
- **3 words**: `gate`, `trigger`, `cv` (temporal types)
- **24 structural symbols**: `@`, `->`, `<-`, `<>`, `{}`, `,`, `()`, `:`, `=`, `[]`, ``` `` ```, `//`, `-`, `_`, `.`, `...`, `!`, `<!`, `#`, `?`, `$`, `&`, `~`, `|`
- **9 flag operators**: comparison `==`, `!=`, `>`, `<`, `>=`, `<=` + calculation `+`, `-`, `=` (`-`/`=` are distinct operators that reuse glyphs also used as structural symbols)
- **7 reserved qualifier keys**: `mode`, `scan`, `weight`, `on_fail`, `tempo`, `meter`, `scale` (per `docs/spec/LANGUAGE.md`; `scan`/`tempo`/`meter` handled in `encoder.js`). `speed` SUPPRIMÉ (décision 2026-06-26) → durée `:` (`{A B}:2`, `A4:1/2`)
- **Double declaration**: each symbol has temporal type + runtime binding (`gate Sa:sc`)
- Silence: `-` in both BPScript and BP3
- Prolongation: `_` in both BPScript and BP3
- Period notation: `.` = equal-duration fragment separator (same as BP3)
- `!` = simultaneous event (any type: trigger, gate, cv, or flag mutation)
- `[]` = engine instructions (BP3): guards, mode, weight, tempo operators (durée = `:`, hors `[]`)
- `()` = runtime instructions: vel, pan, wave, attack, release, filter, etc. (encoded as `_script(CT)`, consumed by a downstream runtime)
- Backticks: code evaluated by the symbol's runtime (implicit) or tagged (`sc:`, `py:`)

### Architecture
- `bp3-engine/` — Submodule: BP3 WASM engine ([roomi-fields/bp3-engine](https://github.com/roomi-fields/bp3-engine))
- `src/transpiler/` — Parser and compiler
  - `tokenizer.js` — Source text → token stream
  - `parser.js` — Tokens → AST (Scene, Directive, Rule, CVInstance, Macro, Polymetry)
  - `encoder.js` — AST → BP3 grammar text + flat alphabet + prototypes + settings
  - `prototypes.js` — Generates BP3 -so. prototype files for terminal durations
  - `index.js` — Facade: `compileBPS(source)` → `{ grammar, alphabetFile, prototypesFile, controlTable, cvTable, errors }`
  - `actorResolver.js` — Resolves actors (alphabet/tuning/octaves bindings) between parser and encoder
  - `libs.js` — Library loader (JSON → controls, symbols, CV objects)
- `src/bpx/` — BPx engine stub (next-generation derivation engine, see BPX specs)
- `lib/` — JSON libraries (controls, alphabets, tunings, filter, routing, etc.)
- `dist/` — BP3 WASM build (bp3.js, bp3.wasm, bp3.data)
- `docs/` — Documentation (5 dossiers par type)
  - `spec/` — Spécifications normatives du langage
    - `LANGUAGE.md` — Spécification complète (vision + langage + compilation BP3)
    - `EBNF.md` — Grammaire formelle (EBNF)
    - `AST.md` — Nœuds AST
  - `design/` — Architecture et design technique
    - `ARCHITECTURE.md` — Pipeline de compilation (source → AST → grammaire BP3) + interface WASM
    - `ACTOR.md` — Acteur = voix (alphabet/tuning/sound/transport/eval), cascade de sortie, voix notes vs code, appareils
    - `PITCH.md` — Résolution pitch 6 couches (actor → alphabet → octaves → temperament → tuning → resolver)
    - `SOUNDS.md` — Résolution terminaux unifiée (spec < CT < CV cascading)
    - `CV.md` — CV/signal objects (ADSR, LFO, ramp)
    - `EFFECTS.md` — Effets et signal processing
    - `HOMOMORPHISMS.md` — Étiquetage post-dérivation
    - `REPL.md` — REPL adapters et backticks
    - `SCENES.md` — Hiérarchie de scènes, scoping flags, @scene/@expose/@map, sys
    - (Les docs du moteur BPx ont migré dans le dépôt BPx : `../BPx/docs/ARCHITECTURE.md` (principes/décisions), `../BPx/docs/ENGINE_SPEC.md` (contrat externe), `../BPx/docs/IMPLEMENTATION.md` (interne))
    - `INTERFACES_BP3.md` — Interface WASM BP3 (in/out)
    - `TEMPORAL_DEFORMATION.md` — Constraint solver, déformation temps réel
  - `reference/` — Guides techniques
    - `WASM_HOWTO.md` — Build et usage WASM
    - `NATIVE_HOWTO.md` — Build et usage natif
    - `BP3_FILE_FORMATS.md` — Formats fichiers BP3
    - `HO_FORMAT.md` — Format homomorphismes
  - `issues/` — Problèmes ouverts
    - `POLYMAKE_STACK.md` — Stack overflow polymétrie imbriquée
    - `RNG_PORTABLE.md` — Portabilité RNG MSVC/glibc
    - `TEMPO_OPS_WASM.md` — Opérateurs tempo `/N` `\N` `_tempo()` : écarts WASM vs natif

### Tour de contrôle inter-projets (OBLIGATOIRE) — outil `tour`
Coordination de l'écosystème (BPScript, BPx, bp3-frontend, runtimes, moteur Bernard) :
dépôt PRIVÉ `/home/romi/dev/bp/hub`. Le protocole est MÉCANISÉ par le CLI `hub/tour`
(plus d'édition markdown des boîtes à la main). Détail : `hub/README.md` (§Le protocole + §Outil tour).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roomi-fields/BPscript](https://github.com/roomi-fields/BPscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
