---
trigger: always_on
description: Este es un **Bootcamp de SQL de Cero a Héroe** estructurado para llevar a
---

# 🤖 Instrucciones para GitHub Copilot

## 📋 Contexto del Bootcamp

Este es un **Bootcamp de SQL de Cero a Héroe** estructurado para llevar a
estudiantes desde cero conocimiento de bases de datos relacionales hasta un
nivel de SQL Developer Junior o Data Analyst Junior.

### 📊 Datos del Bootcamp

- **Duración**: 24 semanas (~6 meses)
- **Dedicación semanal**: 8 horas
- **Total de horas**: ~192 horas
- **Nivel de entrada**: Cero (sin experiencia previa en bases de datos)
- **Nivel de salida**: SQL Developer Junior / Data Analyst Junior
- **Enfoque**: Progresión desde fundamentos absolutos hasta SQL avanzado y
  optimización de consultas
- **Motor principal**: SQLite (fundamentos) → PostgreSQL vía Docker (producción)
- **Otros motores**: MySQL/MariaDB es común en el ecosistema real; los conceptos del
  bootcamp son ~95% transferibles. El contenido central usa exclusivamente
  PostgreSQL/SQLite; las diferencias de sintaxis se documentan con notas puntuales.

---

## 🎯 Objetivos de Aprendizaje

Al finalizar el bootcamp, los estudiantes serán capaces de:

- ✅ Diseñar y crear esquemas de base de datos relacionales normalizados
- ✅ Escribir consultas SQL complejas con JOINs, subqueries y CTEs
- ✅ Utilizar funciones de ventana (window functions) para análisis avanzado
- ✅ Implementar transacciones y garantizar la integridad de los datos (ACID)
- ✅ Optimizar el rendimiento con índices y análisis de planes de ejecución
- ✅ Crear vistas, procedimientos almacenados y funciones
- ✅ Manejar errores y edge cases en consultas reales
- ✅ Modelar datos para casos de uso del mundo real

---

## 📚 Estructura del Bootcamp

### Distribución por Etapas

#### **Etapa 0: Fundamentos de SQL (Semanas 1–8)** — 64 horas

- Qué es una base de datos relacional: tablas, filas, columnas, tipos de datos
- DDL: `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `TRUNCATE`
- DML: `INSERT INTO`, `UPDATE`, `DELETE`
- Consultas básicas: `SELECT`, `FROM`, `WHERE`, `ORDER BY`, `LIMIT`
- Operadores: comparación, lógicos (`AND`, `OR`, `NOT`), `BETWEEN`, `IN`, `LIKE`
- Funciones de agregación: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`
- `GROUP BY` y `HAVING`
- Manejo de `NULL`: `IS NULL`, `IS NOT NULL`, `COALESCE()`, `NULLIF()`
- Constraints: `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, `DEFAULT`, `CHECK`

#### **Etapa 1: SQL Intermedio (Semanas 9–16)** — 64 horas

- `JOIN`s: `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`, `CROSS JOIN`
- `SELF JOIN` para relaciones jerárquicas
- Subqueries: correlacionadas, escalares, en `FROM`, en `WHERE`
- CTEs (`WITH`) y CTEs recursivas
- Funciones de ventana: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LEAD()`, `LAG()`, `FIRST_VALUE()`, `LAST_VALUE()`
- Vistas (`CREATE VIEW`) y vistas materializadas
- Índices: tipos, creación, cuándo usarlos
- Funciones de cadena, fecha/hora y numéricas
- `CASE WHEN` y expresiones condicionales

#### **Etapa 2: SQL Avanzado (Semanas 17–24)** — 64 horas

- Transacciones y propiedades ACID
- Control de concurrencia y niveles de aislamiento
- Procedimientos almacenados y funciones definidas por el usuario
- Triggers
- Optimización de consultas: `EXPLAIN`, `ANALYZE`, plan de ejecución
- Normalización: 1FN, 2FN, 3FN, BCNF, desnormalización estratégica
- Diseño de esquemas para casos reales: OLTP vs OLAP
- PostgreSQL: tipos avanzados (`JSONB`, arrays, `hstore`), extensiones, particionamiento

---

## 🗂️ Estructura de Carpetas

Cada semana sigue esta estructura estándar:

```
bootcamp/week-XX-tema_principal/
├── README.md                 # Descripción y objetivos de la semana
├── rubrica-evaluacion.md     # Criterios de evaluación detallados
├── 0-assets/                 # Diagramas SVG (ER, flujo, índices, etc.)
├── 1-teoria/                 # Material teórico (archivos .md)
├── 2-practicas/              # Ejercicios guiados paso a paso
│   └── ejercicio-XX/
│       ├── README.md         # Instrucciones y pasos
│       ├── starter/
│       │   ├── setup.sql     # Crea tablas e inserta datos de prueba
│       │   └── ejercicio.sql # Consultas comentadas para descomentar
│       └── solution/
│           ├── setup.sql
│           └── ejercicio.sql
├── 3-proyecto/               # Proyecto semanal integrador
│   ├── README.md
│   └── starter/
│       ├── setup.sql         # Esquema genérico adaptable al dominio
│       └── proyecto.sql      # TODOs para implementar
├── 4-recursos/               # Recursos adicionales
│   ├── ebooks-free/
│   ├── videografia/
│   └── webgrafia/
└── 5-glosario/               # Términos SQL clave (A–Z)
    └── README.md
```

### 📁 Carpetas Raíz

- **`assets/`**: Recursos visuales globales (logos, headers, banners)
- **`docs/`**: Documentación general del bootcamp
- **`scripts/`**: Scripts de automatización y utilidades
- **`bootcamp/`**: Contenido semanal del bootcamp

### 🗂️ Orden de Creación de Cada Semana

Al desarrollar el contenido de una nueva semana, seguir **siempre** este orden:

1. `README.md` — Descripción general, objetivos, distribución del tiempo, navegación
2. `rubrica-evaluacion.md` — Tabla de criterios y puntajes
3. `1-teoria/` — Archivos markdown numerados (`01-`, `02-`, …)
4. `0-assets/` — Diagramas SVG vinculados a la teoría
5. `2-practicas/` — Ejercicios con `starter/` + `solution/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ergrato-dev/bc-sql](https://github.com/ergrato-dev/bc-sql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
