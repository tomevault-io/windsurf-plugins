---
trigger: always_on
description: Este es un **Bootcamp de JavaScript Moderno (ES2023)** estructurado para llevar a estudiantes de cero a héroe en desarrollo JavaScript moderno.
---

# 🤖 Instrucciones para GitHub Copilot

## 📋 Contexto del Bootcamp

Este es un **Bootcamp de JavaScript Moderno (ES2023)** estructurado para llevar a estudiantes de cero a héroe en desarrollo JavaScript moderno.

### 📊 Datos del Bootcamp

- **Duración**: 28 semanas (~7 meses)
- **Dedicación semanal**: 8 horas
- **Total de horas**: ~224 horas
- **Nivel de salida**: Desarrollador JavaScript Junior
- **Enfoque**: JavaScript moderno (ES2023) sin historia pre-ES2023
- **Stack**: JavaScript puro, sin frameworks (React/Vue/Angular van en otro bootcamp)

---

## 🎯 Objetivos de Aprendizaje

Al finalizar el bootcamp, los estudiantes serán capaces de:

- ✅ Dominar las características modernas de JavaScript (ES2023)
- ✅ Trabajar con programación asincrónica (Promises, async/await)
- ✅ Manipular el DOM y gestionar eventos de manera efectiva
- ✅ Consumir y trabajar con APIs REST usando Fetch API
- ✅ Aplicar programación funcional y patrones modernos
- ✅ Escribir tests automatizados con Jest
- ✅ Implementar clean code y mejores prácticas
- ✅ Construir aplicaciones completas y complejas con JavaScript puro

---

## 📚 Estructura del Bootcamp

### Distribución por Etapas

#### **Fundamentos Modernos (Semanas 1-12)** - 96 horas

- ES2023 desde el inicio (let/const, arrow functions, destructuring)
- Template literals, spread operator, rest parameters
- Módulos ES (import/export, dynamic imports)
- Arrays modernos (map, filter, reduce, at(), findLast(), toSorted(), toReversed(), with())
- Optional chaining `?.`, nullish coalescing `??`, logical assignment (`??=`, `||=`, `&&=`)
- Clases modernas con campos privados `#` y static blocks
- Numeric separators (`1_000_000`), BigInt

#### **Intermedio (Semanas 13-24)** - 96 horas

- Programación asincrónica (Promises, async/await, top-level await)
- Promise.all(), Promise.race(), Promise.allSettled(), Promise.any()
- Fetch API y trabajo con APIs REST
- Manejo avanzado de errores (Error cause, custom errors)
- DOM moderno y eventos
- LocalStorage, SessionStorage e IndexedDB
- Programación funcional (composición, inmutabilidad)

#### **Avanzado (Semanas 25-28)** - 32 horas

- Testing con Jest
- Patrones de diseño en JavaScript
- Clean code y mejores prácticas
- Debugging avanzado
- Proyectos completos y complejos

---

## 🗂️ Estructura de Carpetas

Cada semana sigue esta estructura estándar:

```
bootcamp/week-XX/
├── README.md                 # Descripción y objetivos de la semana
├── rubrica-evaluacion.md     # Criterios de evaluación detallados
├── 0-assets/                 # Imágenes, diagramas y recursos visuales
├── 1-teoria/                 # Material teórico (archivos .md)
├── 2-practicas/              # Ejercicios guiados paso a paso
├── 3-proyecto/               # Proyecto semanal integrador
├── 4-recursos/               # Recursos adicionales
│   ├── ebooks-free/          # Libros electrónicos gratuitos
│   ├── videografia/          # Videos y tutoriales recomendados
│   └── webgrafia/            # Enlaces y documentación
└── 5-glosario/               # Términos clave de la semana (A-Z)
    └── README.md
```

### 📁 Carpetas Raíz

- **`assets/`**: Recursos visuales globales (logos, headers, etc.)
- **`docs/`**: Documentación general que aplica a todo el bootcamp
- **`scripts/`**: Scripts de automatización y utilidades
- **`bootcamp/`**: Contenido semanal del bootcamp

---

## 🎓 Componentes de Cada Semana

### 1. **Teoría** (1-teoria/)

- Archivos markdown con explicaciones conceptuales
- Ejemplos de código con comentarios claros
- Diagramas y visualizaciones cuando sea necesario
- Referencias a documentación oficial

### 2. **Prácticas** (2-practicas/)

- Ejercicios guiados paso a paso
- Incremento progresivo de dificultad
- Soluciones comentadas
- Casos de uso del mundo real

#### 📋 Formato de Ejercicios (Referencia: week-05)

Los ejercicios son **tutoriales guiados**, NO tareas con TODOs. El estudiante aprende descomentando código:

**README.md del ejercicio:**

```markdown
### Paso 1: Nombre del Concepto

Explicación del concepto con ejemplo:

\`\`\`javascript
// Ejemplo explicativo
const result = data.method(x => x.transform);
\`\`\`

**Abre `starter/index.js`** y descomenta la sección correspondiente.
```

**starter/index.js:**

```javascript
// ============================================
// PASO 1: Nombre del Concepto
// ============================================
console.log('--- Paso 1: Nombre del Concepto ---');

// Explicación breve del concepto
// Descomenta las siguientes líneas:
// const result = data.method(x => x.transform);
// console.log('Resultado:', result);

console.log('');
```

**solution/index.js:**

```javascript
// ============================================
// PASO 1: Nombre del Concepto
// ============================================
console.log('--- Paso 1: Nombre del Concepto ---');

const result = data.method(x => x.transform);
console.log('Resultado:', result);
```

#### ❌ NO usar este formato en ejercicios:

```javascript
// ❌ INCORRECTO - Este formato es para PROYECTOS, no ejercicios
const result = null; // TODO: Implementar
```

#### ✅ Usar este formato en ejercicios:

```javascript
// ✅ CORRECTO - Código comentado para descomentar

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ergrato-dev/bc-javascript-es2023](https://github.com/ergrato-dev/bc-javascript-es2023) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
