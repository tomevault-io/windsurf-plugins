---
trigger: always_on
description: Este repositorio contiene el material educativo para un bootcamp intensivo de **16 semanas** (96 horas, 6h/semana) enfocado en Git y GitHub. El objetivo es formar desarrolladores con dominio completo de control de versiones, colaboración en equipos de desarrollo, y preparación completa para las **4 certificaciones oficiales de GitHub**: Foundations, Actions, Advanced Security y Administration.
---

# GitHub Copilot Instructions - Bootcamp Git/GitHub

## Contexto del Proyecto

Este repositorio contiene el material educativo para un bootcamp intensivo de **16 semanas** (96 horas, 6h/semana) enfocado en Git y GitHub. El objetivo es formar desarrolladores con dominio completo de control de versiones, colaboración en equipos de desarrollo, y preparación completa para las **4 certificaciones oficiales de GitHub**: Foundations, Actions, Advanced Security y Administration.

## Estructura del Proyecto

- `bootcamp/` - Todo el contenido del bootcamp organizado por semanas
- `assets/` - Recursos gráficos globales (banner, logos)
- `scripts/` - Scripts de automatización
- `.github/` - Configuraciones de GitHub y este archivo de instrucciones
- Cada semana tiene su propia carpeta con material teórico y práctico

## Instrucciones para GitHub Copilot

### Estilo y Mejores Prácticas

1. **Documentación en Español**: Todo el contenido debe estar en español, usando un lenguaje claro y profesional
2. **Nomenclatura Técnica en Inglés**: Todos los términos técnicos deben mantenerse en inglés (repository, branch, commit, merge, pull request, etc.)
3. **Markdown Estructurado**: Usar headers apropiados, listas, código con syntax highlighting
4. **Ejemplos Prácticos**: Siempre incluir ejemplos de comandos Git reales y casos de uso
5. **Progresión Didáctica**: El contenido debe seguir una progresión lógica de básico a avanzado
6. **Ejercicios Detallados**: Todos los ejercicios deben incluir comentarios explicativos con: qué se hace, por qué se hace, y para qué sirve

### Estructura de Contenido

Cuando generes contenido para el bootcamp, sigue esta estructura:

````
bootcamp/
├── README.md                     # Índice principal del bootcamp
├── week-01/                      # Git Foundations
├── week-02/                      # Repositories y commits
├── week-03/                      # Branching básico
├── week-04/                      # Merge conflicts
├── week-05/                      # Remote repositories
├── week-06/                      # GitHub features
├── week-07/                      # GitHub Actions fundamentos
├── week-08/                      # CI/CD pipelines
├── week-09/                      # Actions avanzadas
├── week-10/                      # Deployment strategies
├── week-11/                      # Security features
├── week-12/                      # Vulnerability management
├── week-13/                      # Security policies
├── week-14/                      # Enterprise administration
├── week-15/                      # Administration avanzado
└── week-16/                      # Proyecto final + certificaciones
````

### Estructura Interna de Cada Semana

Cada carpeta week-XX/ debe tener esta estructura estandarizada:

```text
week-XX/
├── README.md                     # Descripción general de la semana
├── rubrica-evaluacion.md         # Criterios de evaluación
├── 0-assets/                     # Diagramas y recursos visuales (SVG)
│   └── README.md
├── 1-teoria/                     # Material teórico
│   ├── 01-tema-principal.md
│   ├── 02-segundo-tema.md
│   └── ...
├── 2-practicas/                  # Ejercicios guiados (sin soluciones)
│   ├── README.md
│   ├── ejercicio-01-nombre/
│   │   └── README.md             # Instrucciones paso a paso
│   └── ...
├── 3-proyecto/                   # Proyecto semanal
│   ├── README.md                 # Instrucciones del proyecto
│   ├── starter/                  # Código/archivos iniciales (opcional)
│   └── solution/                 # Solución de referencia (en .gitignore)
├── 4-recursos/                   # Material adicional
│   ├── README.md
│   ├── ebooks-free/              # Libros electrónicos gratuitos
│   ├── videografia/              # Videos y tutoriales
│   └── webgrafia/                # Enlaces y documentación
└── 5-glosario/                   # Términos clave de la semana
    └── README.md
```

### Formato de Lecciones

Cada lección debe incluir:

1. **Objetivos de Aprendizaje**: Qué se espera que el estudiante aprenda
2. **Conceptos Teóricos**: Explicación clara de los conceptos
3. **Comandos y Sintaxis**: Ejemplos prácticos con explicaciones
4. **Ejercicios Guiados**: Paso a paso para practicar
5. **Desafíos**: Ejercicios independientes
6. **Recursos Adicionales**: Enlaces y referencias

### Estándares de Código

- **Comandos Git**: Siempre usar la sintaxis completa antes de mostrar atajos
- **Convenciones**: Seguir Git Flow y Conventional Commits
- **Nomenclatura Técnica**: Mantener términos en inglés (branch, commit, repository, merge, etc.)
- **Comentarios Detallados**: Explicar cada comando con tres niveles:
  - **¿Qué hace?**: Descripción técnica del comando
  - **¿Por qué?**: Razón o contexto para usarlo
  - **¿Para qué sirve?**: Beneficio o resultado esperado
- **Casos de Error**: Incluir cómo manejar errores comunes

### 🔒 Regla de Oro de Versiones (OBLIGATORIA)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ergrato-dev/bc-git-github](https://github.com/ergrato-dev/bc-git-github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
