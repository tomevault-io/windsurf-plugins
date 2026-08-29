---
trigger: always_on
description: Tauri 2-alapú asztali alkalmazás, amely Commodore 64 6502 assembly programok vizuális,
---

# Copilot Instructions — C64 Visual Assembler

## Mi ez a projekt?

Tauri 2-alapú asztali alkalmazás, amely Commodore 64 6502 assembly programok vizuális,
drag-and-drop szerkesztését teszi lehetővé. Nincs UI-framework — csak Vanilla JS, HTML, CSS.
Backend: Rust (Tauri 2), frontend: Vanilla JS/HTML/CSS.

## Plugin integráció (C64AssemblyExpert)

Ez a repo beemelve tartalmazza a C64AssemblyExpert Copilot plugint.

- Agent fájl: `.github/copilot-plugin/agents/c64-assembly-expert.agent.md`
- Skill fájl: `.github/copilot-plugin/skills/c64-assembly-expert/SKILL.md`
- Plugin meta: `.github/copilot-plugin/plugin.json`

Ha a kérdés C64 assembly szakmai tudást igényel (VIC-II, SID, CIA, KERNAL, timing),
akkor ezeket a beemelt plugin fájlokat is vedd figyelembe válaszadás előtt.

---

## Fájlszerkezet

| Fájl | Szerep |
|------|--------|
| `app.js` | Teljes renderer logika (~8 500 sor): mnemonik könyvtár, UI, ASM/monitor generálás, makró expanzió |
| `index.html` | Egyetlen HTML lap; összes UI elem és két `<template>` (block-template, palette-item-template) |
| `style.css` | Teljes stíluslap; CSS custom properties-alapú téma (dark/light) |
| `tauri-bridge.js` | `window.electronAPI` shim — Tauri invoke hívásokat térképez az app.js felé |
| `src-tauri/src/lib.rs` | Tauri backend: VICE/RetroDebugger indítás, fájlmentés/betöltés, SID parse, config |
| `src-tauri/tauri.conf.json` | Tauri konfiguráció, verzió, build targets |
| `package.json` | dev/build scriptek; Tauri CLI |

---

## Kódolási konvenciók

- **Nyelv:** JavaScript (ES2022+, nincs TypeScript, nincs build step a rendererhez)
- **Szintaxis:** `const`/`let`, arrow functionök, template literals, `crypto.randomUUID()`
- **Nincs:** class keyword, import/export (mindent globálisan kezel egy script tag), npm bundler
- **Kommentek:** csak ahol a logika nem egyértelmű; ne kommentáld a triviális sorokat
- **Fordítás:** `translations.hu` és `translations.en` objektumokban; `t(key)` / `tf(key, values)` segédfüggvényekkel; mindig add hozzá az új szövegeket mindkét nyelvhez
- **Mnemonik leírások:** magyar → `mnemonicLibrary[category][i].description`, angol → `mnemonicDescriptionsEn[mnemonic]`

---

## Program adatmodell

Minden blokk (`program[]` tömb eleme) egy plain object:

```js
{
  id: crypto.randomUUID(),       // egyedi azonosító
  category: "Ugrasok",           // mnemonikLibrary kulcs
  mnemonic: "RTS",               // nagybetűs mnemonik string
  operand: "",                   // megjelenített operandus (pl. "#$FF")
  rawOperand: "",                // nyers felhasználói input
  description: "...",            // aktuális nyelven megjelenített leírás
  addressingMode: "implied",     // kulcs az addressingModes-ból
  base: "hex",                   // "hex" | "dec" | "text" | "bytes" | "string"
  validationError: "",           // ha nem üres, a blokk hibás
  // opcionális mezők makrókhoz:
  isTextMacro: true, textX: 0, textY: 0,
  isStringMacro: true, stringAddress: "C000",
  isDataMacro: true, dataAddress: "C000",
  isByteMacro: true,
  isLoopMacro: true, loopReg: "X",  loopCount: "0A", loopLabel: "loop1",
  isNextMacro: true, nextLabel: "loop1", nextReg: "X",
  isLabel: true, labelName: "loop",
  isComment: true, commentText: "...",
  isConstMacro: true, constName: "SCREEN", constValue: 1024,
  // rawOperand tárolja az értéket a blokk aktuális base-ében (pl. "0400" hex-ben)
  // constValue numerikus érték (mindig decimálisan), constName az identifier
  // CONST blokkok felkerülnek a label-táblába assemblelésnél → hivatkozható operandusként
  // Makró paraméterek és invokálás:
  macroParams: "color, count",      // MACRO blokkon: vesszős paraméternévlista
  invokeArgs: "#$07, $0A",          // INVOKE blokkon: vesszős argumentumlista
  isBlankLine: true,                // Expert módban gépelt üres sor — 0 byte, dashed spacer block módban
}
```

---

## Mintaprogram sorrend — TEXT példa

A `loadTextSampleProgram()` helyes sorrendje:
1. `SEI` — megszakítások tiltása
2. `TEXT` blokk(ok) — szöveg kiírása KERNAL CHROUT-on keresztül
3. `RTS` — **mindig az utolsó blokk**; szubrutin visszatérés

> **Fontos:** Az RTS-t soha ne tedd a TEXT makrók elé — azok utána kerülnek a kódba.

---

## Makró expanzió

A `TEXT` makró `compileLineBytes()` híváskor fejlődik ki:
- Direkt screen RAM írás: karakterenkénti `LDA #$screenCode` + `STA $0400+offset` párok
- Koordináta: `textX`, `textY` mezőkből számított `$0400 + (textY * 40) + textX` kezdőcím
- `encodeTextMacro()` → screen code-okba konvertál (nem PETSCII, nem KERNAL CHROUT!)
- Az ASM nézetben deferred `.byte` szekció jelenik meg a kód alján (csak vizuális)

A `STRING` makró: szöveg karakterenként C64 screen code-ban, megadott abszolút memóriacímre (`stringAddress`, pl. `"C000"`). Deferred `.byte` szekció az ASM nézetben + inline kód a memóriatérképen.

A `DATA` makró: nyers byte-ok (`$FF, 169, 0x1A` formátumban), megadott abszolút memóriacímre (`dataAddress`, pl. `"C000"`). Minden byte → `LDA #$xx` + `STA $addr+offset` pár az inline kódban; a raw byte-ok deferred szekciókban is megjelennek.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zstarczali/VisualAssembler](https://github.com/zstarczali/VisualAssembler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
