---
trigger: always_on
description: **Nombre del repo:** `colombia-icons`
---


## 1. Resumen del proyecto

**Nombre del repo:** `colombia-icons`
**Descripción:** Librería de íconos line-icon (outline, minimalista) inspirada en Colombia, distribuible como paquete para React, Angular y Blazor, más una base de íconos genéricos de UI reutilizables en cualquier proyecto.

**Objetivo v1.0:** Publicar 3 paquetes instalables (npm x2, NuGet x1) generados desde una única fuente de verdad en SVG.

---

## 2. Arquitectura del monorepo

```
colombia-icons/
├── icons/
│   └── svg/
│       ├── naturaleza/
│       ├── cultura/
│       ├── gastronomia/
│       ├── mapas/
│       ├── urbano/
│       ├── historia/
│       ├── deportes/
│       └── genericos/
│
├── packages/
│   ├── react/          → npm: @mteherandev/colombia-icons-react
│   ├── angular/         → npm: @mteherandev/colombia-icons-angular
│   ├── blazor/          → NuGet: ColombiaIcons.Blazor
│   └── web-components/  → (opcional, fase 2) <ci-icon name="condor" />
│
├── apps/
│   └── site/             → Vite + React, landing informativo + galería de descarga (SVG/PNG)
│
├── scripts/
│   └── generate/         # genera componentes de cada paquete a partir de icons/svg
│
├── logs/
│   └── <NNN>-<tarea>/    # plan.md + resultado.md por tarea grande (sección 10)
│
├── .github/
│   └── workflows/
│       ├── build-and-test.yml
│       ├── publish-npm.yml
│       └── publish-nuget.yml
│
├── package.json           # raíz, npm workspaces
├── LICENSE
├── README.md              # inglés (default, visible en GitHub/npm)
└── README.es.md           # español
```

**Principio clave:** `icons/svg/` es la única fuente de verdad. Nunca se edita un ícono directamente en `packages/*` ni en `apps/site`; todo se regenera o se lee en vivo desde `scripts/generate/` e `icons/svg/` respectivamente.

**Tooling del monorepo:** npm workspaces (sin Nx/Turborepo), incluyendo `apps/*` además de `packages/*`.

---

## 3. Especificación técnica de cada ícono

- **Grid:** 24x24px
- **Stroke width:** 1.5px, consistente en todo el set
- **Color:** `stroke="currentColor"` (hereda color vía CSS del proyecto consumidor)
- **Estilo:** line icon / outline, esquinas redondeadas (`stroke-linecap="round"`, `stroke-linejoin="round"`)
- **Formato fuente:** SVG optimizado con SVGO (sin metadata, sin IDs redundantes, sin `fill` fijo)
- **Naming de archivo:** `kebab-case.svg` (ej. `sombrero-vueltiao.svg`, `carpeta-abierta.svg`)

---

## 4. Lista de íconos v1.0 (propuesta editable)

### 4.1 Naturaleza (~18)
cóndor, colibrí, ceiba, palma-de-cera, planta-de-cafe, flor-de-mayo, rio, paramo, selva-amazonica, mar-caribe, volcan, mariposa, orquidea, jaguar, delfin-rosado, arrecife-coral, cascada, arbol-tropical, frailejon, sabueso-fino, salto-del-tequendama, sierra-nevada, cocora-valle

### 4.2 Cultura (~12)
sombrero-vueltiao, mochila-wayuu, carnaval-barranquilla, acordeon-vallenato, mola-kuna, ruana, feria-de-las-flores, tejo, chiva-bus, guacharaca, sombrero-aguadeño, guiro

### 4.3 Gastronomía (~14)
arepa, bandeja-paisa, taza-de-tinto, aguardiente, empanada, arepa-de-choclo, sancocho, patacón, arepa-de-huevo, ajiaco, arepa-boyacense, pescado-frito, chicharrón, grano-de-cafe

### 4.4 Mapas y geografía (~7)
silueta-colombia, region-caribe, region-andina, region-pacifica, region-orinoquia, region-amazonica, isla-san-andres

### 4.4.1 Urbano (~2)
bogota-torre, medellin-metro

### 4.4.2 Historia (~3)
cartagena-murallas, cartagena-iglesia, ciudad-perdida

### 4.5 Deportes (~8)
ciclismo, futbol, vuelta-a-colombia, patinaje, tejo-deporte, boxeo, atletismo, natacion

### 4.6 Genéricos de UI (~28)
guardar, eliminar, cancelar, cerrar, estrella, estrella-llena, archivo, carpeta, carpetas, editar, buscar, configuracion, agregar, quitar, check, alerta, informacion, candado, candado-abierto, usuario, calendario, reloj, descargar, subir, compartir, copiar, imprimir, menu-opciones, refrescar

**Total v1.0: ~93 íconos.** (Ajustable — se puede recortar o ampliar por categoría antes de iniciar la generación.)

---

## 5. Empaquetado por framework

### React — `@mteherandev/colombia-icons-react`
- Un componente `.tsx` por ícono (ej. `<Condor size={24} color="currentColor" />`)
- Props: `size`, `color`, `className`, resto de `SVGProps`
- Build con `tsup` o `rollup`, salida ESM + CJS + tipos `.d.ts`
- Tree-shakeable (exports individuales, no un solo bundle gigante)

### Angular — `@mteherandev/colombia-icons-angular`
- Librería generada con `ng-packagr`
- Componente único `<ci-icon name="condor">` con input `name`, o componentes individuales (decidir según preferencia de DX)
- Compatible con Angular standalone components

### Blazor — `ColombiaIcons.Blazor`
- Un componente `.razor` por ícono (ej. `<Condor Size="24" />`)
- Empaquetado como librería `Razor Class Library (RCL)`
- Publicado como paquete NuGet
- Namespace sugerido: `ColombiaIcons.Blazor.Icons`

---

## 6. Sitio web informativo (`apps/site`)

**Propósito:** landing page pública del proyecto — explica qué es colombia-icons y cómo instalarlo en cada framework, y permite explorar y descargar cualquier ícono individualmente en SVG y PNG sin instalar ningún paquete.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mteheran/colombia-icons](https://github.com/Mteheran/colombia-icons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
