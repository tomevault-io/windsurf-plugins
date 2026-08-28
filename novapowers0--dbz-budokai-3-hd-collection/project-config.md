---
trigger: always_on
description: ﻿# DBZ Budokai 3 HD Collection — Contexto del proyecto
---

﻿# DBZ Budokai 3 HD Collection — Contexto del proyecto

> Documento de contexto para agentes/AI. Consolida el estado del proyecto,
> las decisiones tomadas y el trabajo realizado, para no perder información
> entre sesiones.

---

## 0. ✅ MIGRACIÓN A REXGLUE 0.10.0 — COMPLETADA (2026-08-25)

**Leer `docs/MIGRACION_REXGLUE_010.md` ANTES de tocar el SDK.** Documenta la
migración 0.9.0 → 0.10.0 (ya validada en juego).

- **Hecho**: SDK 0.10.0 clonado en `rexglue-sdk-0.10/` (tag v0.10.0) + submodulos;
  build configurado (D3D12+Vulkan+FFX, clang 22, x86-64-v3); 2 bugs del build
  resueltos (libmspack wrappers, ffx_api_dll.rc UTF-16); **parche del runtime
  portado a 0.10** (afs.h/afs.cpp recreados porque 0.10 los eliminó + host_path_*
  refactorizados + 3 cvars dbz1 restaurados); **rexruntime.dll 0.10.0 compilado
  (10933248 B) con los marcadores del parche**.
- **2026-08-25**: **rexgpu-xenos.dll 0.10.0 compilado (6207488 B)** + SDK
  completo; **SDK 0.10 instalado en `rexglue/`** (respaldo 0.9 → `rexglue_0.9/`,
  PACKAGE_VERSION 0.10.0 verificado); **dbz3.exe compilado contra 0.10**
  (17298432 B, Release) con el código generado **regenerado con el codegen 0.10**
  (`dbz3_codegen`, 44 recomp files; el 0.9 usaba `REX_WEAK_FUNC`, eliminado en
  0.10); los fixes del SDK (CallInUIThreadSynchronous timeout + presenter
  pacing) **ya son nativos en 0.10**; `github/patches/` actualizado a 0.10
  (9 archivos + README).
- **✅ VALIDADO EN JUEGO (2026-08-25, usuario)**: los mods `swap_96_on_327`
  (Babidi→Krillin, override simple) y `tex_91` (Gero, texturas) funcionan, y el
  **mid-insert virtual** funciona (mod `sw_vegeta424`, Vegeta armadura, bin
  126976 > to_read 106496 del slot 327). **Migración 0.10.0 COMPLETA.**
- **Estado del SDK**: `rexglue/` = 0.10 (activo). `rexglue_0.9/` = respaldo
  (mantener por si acaso, ya no es el default). `rexglue-sdk/` = fuente 0.9
  (histórico, no tocar).
- **⚠️ Compilar el juego**: el preset `win-amd64-release` resuelve `clang` al
  toolchain retcomm (MinGW, NO compila `rex/chrono/chrono.h`) — pasar SIEMPRE
  `-DCMAKE_CXX_COMPILER="C:/Program Files/LLVM/bin/clang++.exe"` y
  `-DCMAKE_PREFIX_PATH=.../rexglue` (ver MIGRACION §7.3).
- **⚠️ PENDIENTE (cuando se complete el plan general)**: actualizar
  release/README + README para el SDK 0.10 antes de subir a GitHub (el repo
  `github/` aún no se sube; los parches y docs ya están actualizados en local).

---

## 1. QUÉ ES ESTO

Port recompilado a PC de **DBZ Budokai 3 HD Collection (Xbox 360)** usando el
**ReXGlue SDK** (derivado de Xenia). El proyecto incluye el launcher custom
(`src/launcher/`), la lógica de región/mods, y el runtime (rexglue-sdk).

- **dbz3** (este proyecto): Budokai 3 HD Collection
- **dbz1**: `C:\Users\javie\Desktop\PROYECTOS IA\DBZ Budokai HD Collection`
  (proyecto hermano, ya con los fixes de input aplicados)

## 2. UBICACIONES CLAVE

| Ruta | Contenido |
|------|-----------|
| `src/` | Código del launcher y del juego (main.cpp, launcher/, ingame/) |
| `rexglue-sdk/` | SDK fuente (runtime, GPU, filesystem) — compilable |
| `rexglue-sdk/out/build-win-vulkan/` | Build SDK (D3D12+Vulkan+FFX, clang) |
| `out/build/win-amd64-release/` | **Build del juego** (dbz3.exe, DLLs, mods/) |
| `out/build/win-amd64-tracy/` | Build instrumentado Tracy (profiling) |
| `eu/`, `us/` | Assets de región (AFS del juego) |
| `ps2_games/` | AFS de B1, B2, B2V, B3 GH, IW (referencias PS2) |
| `mod center/` | Herramientas de modding (36 programas) |
| `modding resources/` | Documentación + recursos (modelos, listas, arte) |
| `modding resources update/` | Buzón para nuevos archivos del usuario |
| `generated/` | Código recompilado del guest (dbz3_recomp.*.cpp) |
| `docs/` | **Documentación organizada (leer PRIMERO)** — índices en docs/README.md |

## 2.1 DOCUMENTACIÓN (docs/ — LECTURA PRIORITARIA)

El proyecto está documentado en `docs/`. **LEER `docs/README.md` primero** y luego:

- `docs/01_estructura/ARBOL.md` — qué es cada carpeta
- `docs/01_estructura/ESTADO.md` — qué funciona / qué falla (estado actual)
- `docs/02_mods/COMO_HACER_MODS.md` — pipeline de mods (override por entrada)
- `docs/02_mods/MODEL_SWAP.md` — investigación de model swap
- `docs/02_mods/TEXTURAS_MOD.md` — pestaña Texturas del launcher (funcionamiento)
- `docs/03_formatos/AMO_AWO.md` + `BIN_LAYOUT.md` — formato del bin
- `docs/04_herramientas/TOOLS.md` — inventario de herramientas
- `docs/05_build/COMO_COMPILAR.md` — compilar juego/SDK
- `docs/06_limpieza/PLAN_LIMPIEZA.md` — plan de limpieza pendiente

## 3. ESTADO ACTUAL (RESUMEN EJECUTIVO)

- **D3D12 = backend principal** (prebuilt 2.7MB, fluido en 3D)
- **Vulkan = experimental** (marcado en el launcher; 3D lento por el render
  path de Vulkan — 6.5x más lento que D3D12 en IssueSwap)
- **Mando**: `input_backend = "xinput"` (evita cuelgue con RTSS/OBS)
- **Config**: D3D12 + 2x + frame_cap 60 + región us + mod sw_goten_nativo (swap nativo validado)

### 3.1 ESTADO ACTUAL (2026-08-17) — VÍA VALIDADA PARA SWAPS

- **✅ SWAP B3→B3 NATIVO = FUNCIONA** (mod `sw_goten_nativo`): reemplazo del
  bin #AMB COMPLETO (AWO+AZT) del personaje en el AFS. Validado en juego por el
  usuario (calidad excelente, rig 100%, voz/parpadeo ok).
- **El método AFS correcto es MID-INSERT** (bin crece en su slot, entradas

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [novapowers0/DBZ-Budokai-3-HD-Collection](https://github.com/novapowers0/DBZ-Budokai-3-HD-Collection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
