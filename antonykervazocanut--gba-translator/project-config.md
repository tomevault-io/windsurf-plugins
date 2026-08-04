---
trigger: always_on
description: > Pokemon Unbound GBA ROM toolkit: reverse-engineers English + Spanish ROMs and
---

# gba_translator — AI Memory (Codex / AGENTS.md)

> Pokemon Unbound GBA ROM toolkit: reverse-engineers English + Spanish ROMs and
> produces a French ROM translation (CFRU / BPRE01 engine).

## Stack

- Python 3.11+ (main language)
- GNU Make (build automation)
- pytest (unit + integration tests, markers: `emulator`, `rom`, `slow`, `stress`, `benchmark`)
- Playwright + TypeScript (E2E tests against mGBA emulator)
- Vitest (TypeScript unit tests for `emulator-web/`)
- pyyaml ≥ 6.0 (only external Python dep)

## Rules

### Chargement des règles et mémoires

- Codex doit utiliser la compétence auto-découverte
  `.agents/skills/work-on-gba-translator/SKILL.md` pour toute tâche dans ce dépôt.
- Avant de modifier, tester ou déboguer, lire les règles ciblées et les mémoires pertinentes
  indiquées par cette compétence ; ne pas charger l'ensemble des archives sans nécessité.
- Les mémoires sont des retours d'expérience à vérifier contre le code et le `Makefile`
  actuels, pas une autorité supérieure au ticket ou au présent `AGENTS.md`.

### Architecture

- **DRY**: shared Python logic lives in `src/core/`. Never copy functions between scripts.
- No Python scripts at the project root. Scripts belong in `scripts/`; archived ones in `scripts/legacy/`.
- Inputs (`englishrom.gba`, `spanishrom.gba`) are read-only. All derived outputs go under `output/`.
- Write `output_path + ".bak"` before patching any ROM.
- Numbered docs: every file in `docs/` must match `NN_NAME.md`.
- Commit format: `type(scope): description` — no Co-Authored-By trailers, no `--no-verify`.

### Key domain invariants

- CFRU charmap (not ASCII) — see `src/text/charmap_data.py`.
- String terminator: `0xFF`.
- Control codes `FC / FD / F8 / F9 / F7` are multi-byte (2–3 bytes).
- Pointers: 32-bit little-endian, base `0x08000000`.
- Main text region: `0x1F00000–0x1F80000`.
- French chars: é è ê ë à â ç ù û ü î ï ô œ (all supported; ê/ç/ù absent from intro font).
- `FA`/`FB` bytes are outside the translation token queue — do not treat as text.
- `{LV}` = `0x34`; `{COLOR}X` → `FC 01 NN`.

### Testing

**Always run `detect_test_processes` before starting any test suite.**

#### Python (pytest)

- Test files live under `tests/`. Run: `python3 -m pytest tests/ -m "not emulator and not stress and not benchmark" -v`
- Use real binary fixtures (ROM slices in `tests/unit/fixtures/`) — never mock `ROMReader`.
- Follow **AAA** (Arrange / Act / Assert) strictly.
- No `unittest.mock.patch` on internal ROM I/O.
- No `.skip` or `xfail` to silence failures — fix the root cause.
- 100 % pass rate required before committing.

#### TypeScript (Vitest — emulator-web)

- Tests live in `emulator-web/tests/`. Run: `cd emulator-web && npm test`
- Follow **AAA** pattern.
- No `toMatchSnapshot()` for logic tests — write explicit `expect(result).toEqual(...)` assertions.
- One behaviour per `it()` block; `describe` groups are one level deep.

#### E2E (Playwright)

- Tests live in `tests/e2e-playwright/`. Run: `npm run test:e2e`
- Prefer `page.getByRole(...)` and `page.getByText(...)` as locators.
- GBA input goes through the WebSocket Lua bridge, not `page.keyboard`.
- **Never trigger an in-game save during tests** — it overwrites the `.sav` fixture and breaks golden baselines.
- Visual baselines in `tests/e2e-playwright/snapshots/` are committed artefacts; update deliberately.

### Web emulator (emulator-web/)

- Express 5 + `ws` WebSocket bridge to mGBA-WASM.
- Port from `process.env.EMULATOR_PORT ?? 3000` — never hard-coded.
- Route logic in `src/api.ts`; `src/server.ts` only wires them.
- Await mGBA ACK before reading memory after any Lua command.
- `Aaaaaaa` / `Fffffff` player names = naming-screen button mash; treat as no-op.
- Session saves during tests corrupt `.sav` fixtures — always use save-states.

## Skills

Codex auto-découvre les compétences natives dans `.agents/skills/`. Les anciennes procédures
du projet restent dans `.codex/skills/` et doivent être ouvertes explicitement si elles sont
utiles.

| Skill | File | When to Use |
|---|---|---|
| `work-on-gba-translator` | `.agents/skills/work-on-gba-translator/SKILL.md` | Toute tâche : charger les règles et mémoires pertinentes |
| `test-driven-development` | `.codex/skills/test-driven-development/SKILL.md` | Before writing any implementation code |
| `systematic-debugging` | `.codex/skills/systematic-debugging/SKILL.md` | On any bug, test failure, or unexpected behavior |
| `brainstorming` | `.codex/skills/brainstorming/SKILL.md` | Before new features or architectural changes |
| `code-review` | `.codex/skills/code-review/SKILL.md` | After completing a task, before merging |
| `verification-before-completion` | `.codex/skills/verification-before-completion/SKILL.md` | Before any "done" claim or commit |
| `writing-plans` | `.codex/skills/writing-plans/SKILL.md` | After brainstorming approval, before coding |

### Skill priority

1. **Process skills first**: `brainstorming` → `writing-plans` → `test-driven-development`
2. **Quality gates**: `verification-before-completion` before every completion claim
3. **Debugging**: `systematic-debugging` before proposing any fix
4. **Review**: `code-review` after each task or before merge

## Project layout

```
src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AntonyKervazoCanut/gba_translator](https://github.com/AntonyKervazoCanut/gba_translator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
