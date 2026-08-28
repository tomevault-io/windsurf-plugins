---
trigger: always_on
description: > Copyright (c) NovaPowers. Released under the MIT License. Firmado por NovaPowers.
---

# AGENTS.md — DBZ Budokai 1 HD Collection (recompile ReXGlue)

> Copyright (c) NovaPowers. Released under the MIT License. Firmado por NovaPowers.
>
> 2026-08-20. **✅ v0.5.1 El bundle del release ahora incluye la toolchain del
> Model pipeline**: `mod center hd/` (runtime subset: launcher_mod_pipeline.py,
> paths.py, characters_db.py, skin_colors.py, swaps/swap_b1.py,
> conversores/install_b3_to_b1.py + port_b3_to_b1_v2.py,
> analizadores/extract_amb_awo.py) + `tools/` (xbcompress + DLLs). Antes el
> zip NO llevaba "mod center hd" → el Model pipeline (Port/Swap/Scan) fallaba
> para el usuario (no encontraba el script) y parecía que "los mods no
> funcionaban". Ahora el pipeline funciona out-of-the-box (validado: catalog
> 109 B1 + 183 B3, swap/port --dry resuelven la cadena completa desde un
> layout empaquetado limpio). Sigue requiriendo: Python 3 instalado + assets/
> + (para Port B3→B1) el data_cmn.afs del B3. Aplicar/cargar mods ya creados
> sigue siendo autocontenido (sin Python). Detalle: `docs/re/SESION14_BUNDLE_PIPELINE.md`.
> 2026-08-19 (noche). Consolidado: **✅ 4.1 Play crash = thread joinable del
> ModPipeline destruido con el dialog (std::terminate) — FIX recompilado**.
> **✅ 4.2 Dabura→Piccolo válido (reactivado + manifest)**.
> **✅ 4.3 Broly→Nappa = lección 26: tex comprimido 30572 B > slot 1388 (18632 B)
> → truncación LZX → 0xC0000005; pipeline ahora valida y falla claro**.
> 2026-08-20. **✅ v0.4.1 Fix launcher: Model pipeline oculto sin mods** — 
> `DrawModsTab` hacía `return` temprano si la carpeta `mods/` estaba vacía, lo
> que ocultaba el Model pipeline (Port/Swap) → no se podía crear el primer mod
> desde el launcher. Ahora el aviso se muestra pero el pipeline se dibuja
> SIEMPRE (Port/Swap habilitados aunque no exista ningún mod).
> 2026-08-20. **✅ v0.5.0 Vulkan + FSR3 (FidelityFX) integrados** (lección 37):
> la SDK se reconstruyó con `REXGLUE_USE_D3D12=ON REXGLUE_USE_VULKAN=ON
> REXGLUE_ENABLE_FIDELITYFX=ON REXGLUE_FIDELITYFX_BACKEND=dx12`. El runtime
> B1 ahora soporta **D3D12 y Vulkan** (selector en la pestaña Video) y
> **upscalers FidelityFX CAS/FSR/FSR2/FSR3** (selector + calidad FSR). Base
> para la futura versión Linux (Vulkan nativo en la SDK). Detalle:
> `docs/re/SESION13_VULKAN_FSR.md`.
> 2026-08-20. **✅ 5.x Fix piel sin color en ports (Dabura/Buu)**: la causa era
> la piel roja del B3 modelada con MATERIAL (+0x34==5) sobre textura GRIS, no
> con textura roja → el B1 la mostraba descolorida. Nueva opción `--tint-skin`
> en el port que tiñe los bloques DXT3 grises de piel al color objetivo
> (lección 35). Aplicado a Dabura (`--tint-skin 142,9,43`, manifest v1.2).
> **Sistema automático de colores (lección 36)**: `scan_skin_tint.py` detecta
> que personajes tienen piel gris (`skin_grey_majority`); `skin_colors.py` es
> la tabla curada de colores; el port/pipeline usan `--tint-skin auto`
> (resuelve el color desde el label). Cell Jr. = azul/negro, NO verde.
> 2026-08-17. Consolidado v10-v12 + **model swaps B1→B1 100% funcionales** +
> **✅ PORT B3 HD→B1 HD 100% FUNCIONAL (Gero, validado en runtime)** +
> **moveset descartado (lección 13: #ACM no sustituible/generable sin RE)**.
> **✅ SWAP NATIVO CHZ HD COMPLETO → slot TSH validado** (lección 16: bin 352+353,
> 3 AWGs = cuerpo+manos, render perfecto). **Esqueleto B2 PS2 = HD 1:1**
> (Tenshinhan B2 PS2 entry 282/286: 42 labels base idénticos en el mismo orden).
> Formato HD: sec34 = vértice **44B stride**, offsets del header AWG0
> **RELATIVOS al AWG0**. Modelo base PS2 correcto = **B1 PS2**.
> **✅ PORT PS2→HD VIABLE (lecciones 22-24)**: el crash 0xC0000005 era TEX
> MISMATCH (no geometría). Con el par correcto (x_350+x_351) el port CHZ PS2→TSH
> **ENTRA EN COMBATE SIN CRASH** (mod `test_chz_ps2_texfix`). Deforforma por
> decimación voxel + descriptores uniformes → refinar para port fiel.
> **Guía de swaps para el proyecto B3**: `DBZ Budokai 3 HD Collection\mod center
> hd\GUIA_SWAPS_Y_PORTS.md` (principio del swap nativo, B3→B1 ✅, hoja de ruta
> B3→B3 y B1→B3; `awg_to_obj.py` del B3 arreglado 17/08).
> Docs de detalle: `docs/tutoriales/MODEL_SWAPS_METODOLOGIA.md`,
> `docs/re/SESION10_PORT_B3_B1_FUNCIONAL.md`, `docs/re/SESION9_MODEL_SWAPS_B1_B1.md`,
> `docs/re/ANIMACIONES_MOVESETS_HD.md` (set de archivos + movesets),
> `docs/re/SESION11_PORT_PS2_METODOLOGIA.md` (**NUEVA 17/08**: swap nativo CHZ,
> submesh data descifrado, mapeo B2 PS2→HD, hoja de ruta port PS2),
> `docs/planes/PLAN_QOL_WIDESCREEN_LAUNCHER.md` (hoja de ruta QoL/widescreen/launcher),
> `docs/re/SESION7_MESH_GROUP_COMPLETO.md`,
> `docs/re/SESION6_PORT_B2_B1_HD.md`, `docs/planes/PLAN_PORTS_FUNCIONALES.md`.

---

## 🔴 FORMATO HD (RE DEFINITIVA, verificado en TSH nativo slot_2450)

### Vértice sec34 (44B, layout de sesión 5 — CORRECTO)
```
+00 pos.x +04 pos.y +08 pos.z      (floats BE)
+12 weight (float, 0.7/0.8/0.9/1.0)
+16 BONE index (u32, VÁLIDO 1-34)
+20 nrm.x +24 nrm.y +28 nrm.z
+32 0xFFFFFFFF
+36 blend/scale
+40 uv
```
`n_sec = sec_size//44`. TSH nativo = **4272 verts**.

### Offsets del header AWG0 (+0x50) — RELATIVOS al AWG0 (NO absolutos)
```
+0x28 sec_off  → sec_abs  = AWG0 + val    (0xB20+0x24D0 = 0x2FF0)
+0x2C sec_size → n_sec = sec_size//44
+0x30 post_off → post_abs = AWG0 + val    (0xB20+0x30310 = 0x30E30 = sec+sec_size ✓)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [novapowers0/DBZ-Budokai-1-HD-Collection](https://github.com/novapowers0/DBZ-Budokai-1-HD-Collection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
