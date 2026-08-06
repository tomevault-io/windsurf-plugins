---
trigger: always_on
description: Poder. Eficiencia. Iberofonía. Interlingua.
---

# Falcato — AGENTS.md

## Filosofía del proyecto

Poder. Eficiencia. Iberofonía. Interlingua.

Falcato NO es una traducción de Rust al español. Es un lenguaje de bajo nivel
*construido desde cero* sobre **Cranelift** — apuesta estratégica, no temporal.

El sistema de tipos y semántica están diseñados aprovechando las dimensiones
gramaticales del español que el inglés no tiene: género, tiempos verbales,
ser/estar, subjuntivo, prefijos productivos, voz pasiva/activa, compuestos
aglutinantes.

### Visión estratégica

Falcato + Cranelift + WASM = **toolchain nativa para código generado por IA**.

- **Falcato**: lenguaje interlingua entre humanos (español), LLMs (lenguaje natural
  estructurado, baja ambigüedad), y máquinas (WASM/Cranelift)
- **Cranelift**: compilación ultra-rápida (JIT + AOT), ideal para ciclos LLM →
  código → compilar → ejecutar → depurar
- **WASM**: sandbox nativo para ejecución segura de código no confiable generado
  por IA
- **Bytecode Alliance** (Mozilla, Fastly, Intel, Arm, Google, Microsoft, Shopify):
  alineación natural — necesitan ejecución rápida y segura de código arbitrario

**Velocidad de compilación > velocidad de ejecución optimizada.**
**Seguridad del sandbox > control total del hardware.**
**Lenguaje cercano al humano > notación matemática.**

Cranelift no es "lo que tocó" — es el backend oficial y estratégico.
Contribuimos activamente a su ecosistema y roadmap. No planeamos migrar a LLVM.
Si Cranelift necesita features para lenguajes nativos AOT, las implementamos.

## Reglas de diseño

1. **Cada palabra reservada debe aportar semántica, no solo sintaxis.**
   No es `if` → `si`. Es usar el modo subjuntivo para codificar incertidumbre.
   No es `fn` → `función`. Es usar el tiempo verbal para codificar modo de ejecución.

2. **Cero abstracciones gratuitas.** Si una feature no se puede implementar
   con costo cero en runtime, no pertenece al núcleo del lenguaje.

3. **Explotar, no imitar.** Las features del español (género, ser/estar, etc.)
   deben traducirse en *garantías de compilación*, no en azúcar sintáctico.

4. **Iberofonía no es nacionalismo.** Es explorar si un idioma distinto al inglés
   puede aportar algo nuevo a la ingeniería de lenguajes de programación.

## Los 5 Pilares

| # | Pilar | Esencia | Estado |
|---|-------|---------|--------|
| I | **Género = Ownership** | `el`=owned mutable, `la`=borrowed immutable, `un`=option | ✅ Implementado |
| II | **Ser/Estar = Const/Mut** | `es`=identidad permanente, `está`=estado temporal | ✅ Implementado (base) |
| III | **Tiempos = Modos ejecución** | Presente=sync, Futuro=async, Subjuntivo=fallible | ✅ Subjuntivo + Futuro (18A MVP) |
| IV | **C ABI por defecto** | Layout C, calling C, mangling off | ✅ Implementado |
| V | **Prefijos semánticos** | `re-`=retry, `des-`=free, `pre-`=comptime | 📝 Documentados; `des-` parcial vía FFI manual |

### Estrategia de backend

**Cranelift es el backend oficial y estratégico.** No es temporal ni migraremos a LLVM.
Si Cranelift necesita features (mejor cold branch hinting, debug info AOT, etc.),
las implementamos nosotros y contribuimos upstream. Bytecode Alliance comparte
nuestra visión: código generado por IA necesita compilación rápida y ejecución segura.

### Innovación Fase 12: Concordancia de Posesión

Extensión del Pilar I con **tipos afines** como base teórica:
- `el` = affine (usar 0 o 1 veces, owned)
- `la` = no-lineal (usar N veces, borrowed)
- `los`/`las` = shared ownership (reference-counted)
- Lifetimes léxicos: `&dato Texto` en vez de `&'a T`
- Borrow checker **gradual** (Nivel 0 permisivo → Nivel 2 estricto)
- Diseño completo: `docs/diseno_ownership.md`

## Innovación Implementada: Concordancia Lingüística

Aprovechamos que en español los adjetivos **concuerdan** en género y número con
el sustantivo. En Falcato, los valores deben "concordar" en tipo, ownership y estado.

**Errores intuitivos para hispanohablantes:**
```
[T001] test.fc:4:8: Disconcordancia de tipo: 'a' es 'Entero32' pero se declaró como 'Booleano'
       │ sugerencia: Cambia el tipo a 'Entero32' o el valor

[O001] test.fc:5:5: 'constante' no es mutable: se declaró con 'la' (inmutable)
       │ sugerencia: Usa 'el constante' para hacerlo mutable
```

## Day-0: Decisiones arquitectónicas vinculantes

### C ABI por defecto (no negociable)
- Layout de structs = C layout (`repr(C)` es el default)
- Calling convention = SystemV/C
- Name mangling = desactivado (símbolos literales)
- Salida `.o` compatible con `gcc`/`clang`/`link.exe`

### Span en cada nodo del AST (no negociable)
- `Span { inicio: Posicion, fin: Posicion, archivo: Arc<str> }`
- `Posicion { linea: u32, columna: u32, offset: u32 }`
- Sin Span no hay errores con ubicación ni LSP futuro

### Errores en español con códigos (no negociable)
- Formato: `[T001] archivo.fc:7:12: mensaje`
- Categorías: S (sintaxis), T (tipo), O (ownership), C (FFI), M (módulos), I (interno), W (warning)
- Sugerencia opcional en cada error

## Stack técnico

- **CLI:** `clap` 4.5 (Rust)
- **Lexer:** `logos` 0.14 — errores léxicos reportados con span real
- **Parser:** descendente manual modular (Pratt parser), recovery de errores, spans reales
- **AST:** Propio con Span obligatorio, métodos `span()` en nodos
- **Semántica:** "Concordancia Lingüística" — tipos + ownership + bounds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CerebroCanibalus/falcato](https://github.com/CerebroCanibalus/falcato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
