---
trigger: always_on
description: > Instrucciones para agentes AI que trabajan en este repositorio.
---

# AGENTS.md - Code 301 - Professional Software Development (`code-301`)

> Instrucciones para agentes AI que trabajan en este repositorio.
> Compatible con: Claude Code, GitHub Copilot, Cursor, Gemini CLI, Windsurf, Codex.

---

## Proyecto

Repositorio del curso **Code 301 - Professional Software Development** de EnterTechSchool (código interno: `code-301`). Contiene el currículo completo del curso de desarrollo web full stack: READMEs, laboratorios, slides, guías de facilitador y rúbricas.

- **Total de clases:** 26 sesiones
- **Estructura:** 5 módulos técnicos (4 clases c/u) + 1 proyecto final (6 clases)
- **Duración por clase:** 150 min síncronos + 2 h asíncronas
- **Modalidad:** Blend (teoría + práctica)
- **Stack:** React + Vite (M1–M3), PostgreSQL + Supabase (M3), Next.js 15 + TypeScript + Prisma (M4–M6), NextAuth.js v5 (M5). **Sin Express.**
- **Gestor de paquetes:** `pnpm` en todo el curso (reemplaza a `npm` por seguridad de cadena de suministro y rendimiento)
- **IA:** Integrada transversalmente en módulos 1-5
- **Prerrequisito:** Code 201 (HTML/CSS, JavaScript, DOM, Git)

---

## Fuente de Verdad

**`README.md` es la única fuente de verdad.** Contiene:

- Estructura del curso (módulos, clases, proyectos integradores)
- Contenido clave por clase y enfoque de IA
- Sistema de evaluación (rúbricas, labs calificados, proyecto final)
- Tecnologías por módulo
- Prerrequisitos heredados de Code 201

**Regla:** Leer SIEMPRE `README.md` antes de generar o modificar contenido. Nunca hardcodear información que ya está en el syllabus. Si hay conflicto entre un archivo de clase y `README.md`, el syllabus gana.

---

## Estructura del Repositorio

```
├── README.md                        # Syllabus (fuente de verdad)
├── AGENTS.md                        # Este archivo
├── CLAUDE.md                        # Configuración Claude Code
├── curriculum/
│   └── class-{01..26}/              # 26 clases del curso
│       ├── README.md                # Teoría para el estudiante
│       ├── lab/README.md            # Laboratorio paso a paso
│       ├── lab/rubric.md            # Rúbrica (solo clases 4, 8, 12, 16, 20)
│       ├── slides/README.md         # Presentación reveal.js
│       ├── facilitator/README.md    # Guía pedagógica
│       ├── test/README.md           # Solo en clases 8 y 20 (test de M2 y M5)
│       ├── test/questions.md        # 5 preguntas que cubren las 4 clases del módulo
│       ├── infographic/index.html   # Infografía HTML (opcional)
│       └── infographic/image-prompts.md
│   └── module-{1..6}/
│       └── MODULE-PLAN.md           # Blueprint del módulo (pre-aprobación)
├── code-challenges/                 # Retos asíncronos
├── context/                         # Documentación técnica de referencia
├── configs/                         # Configuraciones de proyecto
├── _entregas/                       # Entregables del estudiante
├── tests/                           # (Legacy / placeholders — pendiente de limpieza)
└── .claude/skills/                  # Skills de generación de contenido (junctions)
```

### Mapa clase → módulo

| Clases | Módulo | Nombre | Proyecto integrador |
|--------|--------|--------|---------------------|
| 01–04 | Módulo 1 | React Fundamentals | Agenda de Contactos (vista y navegación) |
| 05–08 | Módulo 2 | Data Fetching & Forms | Agenda de Contactos + consumo de API + CRUD |
| 09–12 | Módulo 3 | Bases de Datos SQL con Supabase | Movie Match (schema + RLS + cliente React) |
| 13–16 | Módulo 4 | Backend Full-Stack con Next.js | Movie Match Full-Stack (Next.js + Prisma sobre la misma DB) |
| 17–20 | Módulo 5 | Autenticación, Roles y Middleware | MinCommerce con NextAuth.js v5 |
| 21–26 | Módulo 6 | Proyecto Final | Aplicación libre (individual o grupal) |

---

## Convenciones

### Idioma y Encoding

- **Idioma:** Español latinoamericano
- **Encoding:** UTF-8 puro (tildes directas: á, é, í, ó, ú, ñ, ü)
- **Signos:** ¿?, ¡! (nunca omitir el signo de apertura)
- **Corrupción:** Si aparecen `�` o `ó`, el archivo está corrupto — regenerar

### Terminología técnica de desarrollo web

- **Conservar en inglés:** React, JSX, props, state, hooks, useState, useEffect, Context API, Router, async/await, fetch, REST, API, endpoint, middleware, MVC, ORM, schema, query, foreign key, join, OAuth, JWT, Next.js, App Router, Route Handler, Server Action, Server Component, Client Component, TypeScript, interface, deploy, build, bundler, pnpm, package, Supabase, Row Level Security (RLS), Prisma, prop drilling, lifting state, controlled component.
- **Traducir al español:** Explicaciones pedagógicas, objetivos de aprendizaje, instrucciones del lab, prompts para el estudiante.
- **Formato:** Primera vez que aparece un acrónimo explicarlo: "ORM (Object-Relational Mapping)". Las siguientes apariciones, solo el acrónimo.

### Enlaces Externos (GitHub Pages / Kramdown)

```markdown
# Externos: SIEMPRE con {:target="_blank"}
[React Docs](https://react.dev/){:target="_blank"}

# Internos: NUNCA con target
[Clase 01](../class-01/)
[Sección](#seccion)
```

### Emojis

Estratégicos en headers para escaneo visual. No decorativos ni excesivos.

---

## Tono por Tipo de Archivo

| Archivo | Audiencia | Tono | Persona |
|---------|-----------|------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [entertechschool/code-301-guide](https://github.com/entertechschool/code-301-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
