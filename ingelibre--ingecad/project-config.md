---
trigger: always_on
description: **Autor:** Marco Sumari Tellez · **Licencia:** GPL-3.0-or-later · **Repo destino:** `github.com/ingelibre/ingecad`
---

# IngeCAD — CAD 2D libre estilo AutoCAD clásico

**Autor:** Marco Sumari Tellez · **Licencia:** GPL-3.0-or-later · **Repo destino:** `github.com/ingelibre/ingecad`
**Hermanos:** [IngeTrazo](../ingetrazo/) (modelador 3D/BIM) · [IngePresupuestos](../ingepresupuestos-pyside6/) (presupuestos)

> Plan fundacional definido el 2026-07-16 (conversación estratégica completa en memoria de Claude: `[[project-ingecad-nuevo-hermano-2d]]`). Este archivo guarda el **rumbo** (visión, principios, fases + DoD); el registro de lo hecho vive en los commits de git. No duplicar acá lo que git ya registra.

---

## 🧭 Visión de producto

**Qué es:** el "AutoCAD LT libre" para Linux — visor/editor 2D de DWG/DXF para el ingeniero que viene de AutoCAD: dibujo rápido con comandos de teclado idénticos a AutoCAD, interfaz clásica pre-ribbon, y apertura **fiel** de los DWG que mandan los colegas. Con capacidades de elevación para el oficio civil (puntos topográficos con cota, terrenos con pendiente, carreteras) — datos de elevación, NO modelado 3D.

**Qué NO es:** no es un clon de AutoCAD feature-por-feature (esa es la receta para nunca shippear — lección de IngeTrazo). AutoCAD tiene cientos de funciones que ni Marco usa. El scope es SU flujo real: **línea, círculo, polilínea, polígono, bloques, capas, hatch, trim, offset, extend, move/copy/rotate, zoom, capas, puntos topográficos, área, imprimir a escala.** Nada más hasta que duela.

**El filtro maestro (heredado del ecosistema):** *"¿le sirve al ingeniero que abre el plano de un colega y dibuja rápido con el teclado?"* Si una feature no pasa ese filtro, no entra.

**La tesis de adopción:** la migración desde AutoCAD debe ser **cero fricción de memoria muscular** — mismos aliases (`M`+Enter = MOVE), misma command line, misma selección ventana/crossing, mismos osnaps. El usuario tipea lo de siempre y funciona.

**Reparto con IngeTrazo (no competir contra el hermano):** IngeCAD = el plano 2D que se firma e imprime (lindero, cuadro de coordenadas, planta). IngeTrazo = el 3D (terreno, modelo, BIM → metrado → IngePresupuestos). Mismo CSV topográfico entra a ambos. Puente entre ellos: DXF.

---

## 📐 Principios arquitectónicos (NO negociables)

1. **El documento ezdxf ES el modelo.** No inventar un modelo de datos propio: se editan las entidades ezdxf directamente (envueltas en Commands) y se guarda con ezdxf. Esto garantiza la propiedad más valiosa del producto: **round-trip conservador** — todo lo que IngeCAD no entiende (proxies de Civil 3D, XDATA, diccionarios, 3DSOLID) se preserva **intacto** al reescribir. "Le devolví el plano sano al colega" es la promesa central.
2. **DWG jamás se parsea dentro del app.** Tres satélites como procesos externos (patrón skp2dae de IngeTrazo): **LibreDWG** (GPL-3, embebible y EMBEBIDO — lectura hasta r2018 de fábrica, escritura r2000), **Open CAD Studio** (MIT, Rust, de César/acadrust; `OpenCADStudio --export src dst`; detectado si está instalado — export r2018 y lector/escritor de respaldo donde no hay LibreDWG, p. ej. Windows; conserva la versión DXF de origen, por eso `_upgrade_dxf` sube a AC1032 antes de pedir r2018) y **ODA File Converter** (freeware propietario, instalación opcional de un clic, NUNCA bundlear — da export r2013/r2018). El usuario abre `.dwg` con doble clic y nunca ve el DXF intermedio.
3. **Coordenadas verdaderas float64 en el modelo; float32 solo en el render.** Los planos reales vienen en UTM (~500 000 Este). DXF/ezdxf guardan doubles — el archivo nunca pierde precisión. El viewport resta un **origen de vista** (centro del dibujo) antes de subir a GPU y lo suma al leer el mouse. El gotcha ya se sufrió en IngeTrazo (`SceneDatum`); acá el fix vive solo en el render.
4. **Toda mutación pasa por Command** (undo/redo exacto) y **todo comando es una acción headless** (`actions.move(...)`, no lógica pegada al evento de teclado/mouse). Es el invariante AI-native del ecosistema, y de paso da macros/scripts gratis — a los usuarios de AutoCAD (LISP) les importa.
5. **2D con Z latente.** DXF es 3D nativo: toda entidad tiene Z y OCS (que un visor correcto debe manejar igual — círculos con extrusión invertida existen en planos reales). El modelo conserva Z siempre; la cámara es ortográfica en planta. Agregar vista isométrica después = solo display (la `OrbitCamera` de IngeTrazo está a un copy de distancia). **3DSOLID (ACIS) jamás se interpreta** — se preserva intacto en el round-trip.
6. **Linux/Wayland first.** Heredar los gotchas resueltos de IngeTrazo: `glClear` explícito en `paintGL`, FBO propio si hace falta, DPR físico vs lógico, re-establecer estado GL tras QPainter, MSAA en el FBO de escena. Windows después, con el pipeline CI ya probado (spec PyInstaller + Inno) — pero ninguna decisión puede ROMPER Windows, solo diferirlo.
7. **Interfaz clásica pre-ribbon, para siempre.** Barra de menús (Archivo/Edición/Ver/Insertar/Formato/Herramientas/Dibujo/Acotar/Modificar) + toolbars acoplables (Draw a la izquierda, Modify a la derecha) + **ventana de comandos abajo** (historial + prompt) + status bar con toggles (FORZC/REJILLA/ORTO/POLAR/REFENT). Fondo de modelo oscuro por defecto. El ribbon no existe ni existirá.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ingelibre/ingecad](https://github.com/ingelibre/ingecad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
