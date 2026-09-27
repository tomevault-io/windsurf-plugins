---
trigger: always_on
description: Lenguaje de bajo nivel *construido desde cero* sobre **Cranelift** (apuesta estratégica, no temporal). NO es traducción de Rust. Es **gramatipado** (la gramática española es el sistema de tipos) y **morfosemántico** (la morfología porta significado de máquina): género, tiempos verbales, ser/estar, subjuntivo.
---

# Falcato — AGENTS.md

## Filosofía
Lenguaje de bajo nivel *construido desde cero* sobre **Cranelift** (apuesta estratégica, no temporal). NO es traducción de Rust. Es **gramatipado** (la gramática española es el sistema de tipos) y **morfosemántico** (la morfología porta significado de máquina): género, tiempos verbales, ser/estar, subjuntivo.

**Visión:** Falcato + Cranelift + WASM = toolchain nativa para código generado por IA. **Velocidad de compilación > velocidad de ejecución optimizada.**

## Los 5 Pilares
| # | Pilar | Esencia | Estado |
|---|-------|---------|--------|
| I | Género = Ownership | `el`=owned mut, `la`=borrowed inmut, `un`=option | ✅ |
| II | Ser/Estar = Const/Mut | `es`=permanente, `está`=temporal | ✅ |
| III | Tiempos = Modos | Presente=sync, Futuro=async, Subjuntivo=fallible | ✅ |
| IV | C ABI por defecto | Layout C, calling C, mangling off | ✅ |
| V | ~~Prefijos semánticos~~ | ~~`re-`=retry~~ | ⛔ Retirado 2026-08-03 |

## Day-0 (no negociable)
- **🚨 TODO EN ESPAÑOL**: lenguaje, errores, CLI, docs. Excepciones: términos técnicos sin traducción (Cranelift, CLIF, JSON, LSP, WASM).
- **C ABI por defecto**: layout C, SystemV, mangling off, salida `.o`
- **Span en cada nodo AST** — sin span no hay LSP
- **Errores en español con códigos** `[T001] archivo.fc:7:12: mensaje` — S/T/O/C/M/I/W
- **Documentar al agente**: cambios grandes → `falcato.md` + skill `falcato-language` en la misma tanda
- **🚨 SEGURIDAD CRÍTICA**: red/sistema/entrada externa → revisión minuciosa antes de mergear
- **NINGUNA ETIQUETA CAMBIA SEMÁNTICA** — etiqueta solo decide CÓMO se produce el binario
- **`--destino` es la ÚNICA etiqueta de plataforma** — el `.fc` nunca sabe dónde corre
- **Código portable o no compila**: builtin sin impl para target = error
- **Impls juntas**: Windows + POSIX en la misma tanda
- **VERSIONADO**: `MAYOR.menor.parche` — Bump en `Cargo.toml` + tag `vMAYOR.menor.parche`
- **RELEASES EN ESPAÑOL** y **NOVEDADES POR EFECTO** (➕/🔧/🔁, no por fase)

## Problemas abiertos de diseño de lenguaje

**Visión guía:** potencia de Rust · facilidad de Go · morfología española como superpoder para LLMs.

Falcato no traduce Rust ni clona Go. Es **gramatipado y morfosemántico**: la gramática española ES el sistema de tipos, y la morfología verbal codifica modos de cómputo (presente=sync, futuro=async, subjuntivo=fallible). Eso da a los LLMs una propiedad única — el código se lee como español natural.

Estamos en **"pasos de bebé"**: todavía hay tensiones de diseño que debemos resolver ANTES de comprometernos. Cada decisión acá es **casi irreversible** (cambia el "lenguaje sentido" durante años). Mejor resolver lento y bien que rápido y mal.

### Tensiones vivas

| # | Tensión | Estado | Próximo paso |
|---|---------|--------|--------------|
| **P-001** | Stdlib: por tipos (Go: `texto`, `archivo`, `red`) vs por intención (verbos: `hacer.archivo.leer`) | ✅ **RESUELTO 2026-08-28**: tipos fragmentados + verbos consistentes + namespaces explícitos (`::`) + conjugación como azúcar | Implementar en 0.8.0 |
| **P-002** | Sintaxis namespace: `.` (colisión con métodos/campos) vs `::` (Rust-like) vs `snake_case` | ✅ **RESUELTO 2026-08-28**: `::` (evidencia LLMs; coma descartada) | Implementar en 0.8.0 |
| **P-003** | Auto-import: prelude pequeño (Rust) vs todo-std auto (Python) | 🟡 Vinculado a P-001 | Diferir a 1.0 |
| **P-004** | Doble API método/función: `t.contiene(sub)` vs `texto.contiene(t, sub)` | 🟡 Diseño inestable | Formalizar regla antes de 1.0 |
| **P-005** | Builtins inflados: 30+ en Capa 1 (FFI) vs reducir a ~15 | 🟢 Activo | Migración gradual 0.8.x |
| **P-006** | Default `Entero` = 32 (rompe ABI) vs 64 | 🟡 Diferido | RFC 0.8.0 con aliases |
| **P-007** | Keyword renames: `apodo`/`alias`, `rasgo`/`protocolo`, `retornar`/`devolver` | 🟡 Diferido | Requiere `MAYOR` (1.0) |
| **P-008** | Mensajes de error: `T001 disconcordancia` vs `no coincide` | 🟡 Diferido | Decisión de estilo 0.8.0 |

#### Regla de migración de builtins (P-005)

**Regla:** Builtins de **solo lectura** → Falcato puro. Builtins de **creación/escritura** → C.

| Categoría | Builtins | ¿Por qué? |
|-----------|----------|------------|
| **Solo lectura** | `contiene`, `empieza_con`, `termina_con`, `longitud` | Solo comparan bytes, devuelven valores simples |
| **Creación** | `mayusculas`, `minusculas`, `recortar`, `reemplazar` | Necesitan malloc para crear nuevo Texto |
| **I/O** | `archivo_*`, `tcp_*`, `http_*` | Syscalls del SO |
| **Hardware** | `lienzo_*`, `imagen_*`, `audio_*` | Win32/X11 APIs |

**Implementación Falcato puro:**
```falcato
// Solo lectura — loop + comparación de bytes
el función texto_contiene(la t: Texto, la sub: Texto) -> Booleano {
    el largo_sub: Entero32 = texto_longitud(sub);
    si largo_sub == 0 { retornar verdadero; }
    el largo_t: Entero32 = texto_longitud(t);
    si largo_sub > largo_t { retornar falso; }
    el limite: Entero32 = largo_t - largo_sub;
    el i: Entero32 = 0;
    mientras i <= limite {
        el encontrado: Booleano = verdadero;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CerebroCanibalus/Falcato](https://github.com/CerebroCanibalus/Falcato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
