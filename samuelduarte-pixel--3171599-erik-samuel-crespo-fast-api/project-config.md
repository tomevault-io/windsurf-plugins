---
trigger: always_on
description: Este es un **Bootcamp de FastAPI Zero to Hero** estructurado para llevar a estudiantes de cero a héroe en desarrollo de APIs modernas con Python y FastAPI.
---

# 🤖 Instrucciones para GitHub Copilot

## 📋 Contexto del Bootcamp

Este es un **Bootcamp de FastAPI Zero to Hero** estructurado para llevar a estudiantes de cero a héroe en desarrollo de APIs modernas con Python y FastAPI.

### 📊 Datos del Bootcamp

- **Duración**: 16 semanas (~4 meses)
- **Dedicación semanal**: 6 horas
- **Total de horas**: ~96 horas
- **Nivel de salida**: Desarrollador Backend Junior (FastAPI)
- **Enfoque**: FastAPI moderno con Python 3.14+
- **Stack**: FastAPI 0.128+, SQLAlchemy 2.0.46+, Pydantic 2.12+, SQLite/PostgreSQL 17+, Docker 27.5+

---

## 🎯 Objetivos de Aprendizaje

Al finalizar el bootcamp, los estudiantes serán capaces de:

- ✅ Dominar los fundamentos de Python necesarios para desarrollo backend
- ✅ Construir APIs RESTful completas con FastAPI
- ✅ Implementar validación de datos robusta con Pydantic v2
- ✅ Trabajar con bases de datos usando SQLAlchemy (sync y async)
- ✅ Implementar autenticación y autorización (JWT, OAuth2)
- ✅ Escribir tests automatizados con pytest
- ✅ Documentar APIs automáticamente (OpenAPI/Swagger)
- ✅ Desplegar aplicaciones con Docker y servicios cloud
- ✅ Aplicar clean code, patrones de diseño y mejores prácticas
- ✅ Construir proyectos completos listos para producción

---

## 📚 Estructura del Bootcamp

### Distribución por Etapas

#### **Fundamentos (Semanas 1-4)** - 24 horas

- Introducción a Python moderno (3.14+)
- Type hints y tipado estático
- Programación asíncrona (async/await)
- Introducción a FastAPI y primeras APIs
- Pydantic v2: validación y serialización
- Rutas, parámetros y query strings

#### **Intermedio (Semanas 5-10)** - 36 horas

- Modelos de datos complejos con Pydantic
- SQLAlchemy ORM y Alembic (migraciones)
- Operaciones CRUD completas
- Relaciones entre modelos (1:N, N:M)
- Manejo de errores y excepciones personalizadas
- Middleware y dependencias
- Background tasks y eventos
- **Evolución arquitectónica: MVC → Hexagonal**

##### 🏗️ Progresión Arquitectónica (Semanas 5-10)

| Semana | Arquitectura | Componentes Nuevos |
|--------|--------------|-------------------|
| 05 | Monolítico simple | SQLAlchemy básico, todo en endpoints |
| 06 | + Service Layer | Services para lógica de negocio |
| 07 | + Repository Pattern | Repositories para acceso a datos |
| 08 | MVC/Capas completo | Routers → Services → Repositories |
| 09 | + Ports & Adapters | Interfaces, inversión de dependencias |
| 10 | Hexagonal completo | Domain, Application, Infrastructure |

#### **Avanzado (Semanas 11-14)** - 24 horas

- Autenticación JWT y OAuth2
- Autorización basada en roles (RBAC)
- Testing con pytest y pytest-asyncio
- WebSockets y Server-Sent Events
- Rate limiting y seguridad
- Logging y monitoreo

#### **Producción (Semanas 15-16)** - 12 horas

- Docker y containerización
- CI/CD básico (GitHub Actions)
- Deployment (Railway, Render, AWS)
- Proyecto final integrador
- Mejores prácticas de producción

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
├── 3-proyecto/               # Proyecto semanal integrador con carpeta solution (oculta para el repo)
├── 4-recursos/               # Recursos adicionales
│   ├── ebooks-free/          # Libros electrónicos gratuitos
│   ├── videografia/          # Videos y tutoriales recomendados
│   └── webgrafia/            # Enlaces y documentación
└── 5-glosario/               # Términos clave de la semana (A-Z)
    └── README.md
```

### 📁 Carpetas Raíz

- **`_assets/`**: Recursos visuales globales (logos, headers, etc.)
- **`_docs/`**: Documentación general que aplica a todo el bootcamp
- **`_scripts/`**: Scripts de automatización y utilidades
- **`bootcamp/`**: Contenido semanal del bootcamp

---

## 🎓 Componentes de Cada Semana

### 1. **Teoría** (1-teoria/)

- Archivos markdown con explicaciones conceptuales
- Ejemplos de código con comentarios claros
- Diagramas y visualizaciones cuando sea necesario
- Referencias a documentación oficial de FastAPI

### 2. **Prácticas** (2-practicas/)

- Ejercicios guiados paso a paso
- Incremento progresivo de dificultad
- Soluciones comentadas
- Casos de uso del mundo real

#### 📋 Formato de Ejercicios

Los ejercicios son **tutoriales guiados**, NO tareas con TODOs. El estudiante aprende descomentando código:

**README.md del ejercicio:**

```markdown
### Paso 1: Crear endpoint GET básico

Explicación del concepto con ejemplo:

\`\`\`python
# Ejemplo explicativo
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
\`\`\`

**Abre `starter/main.py`** y descomenta la sección correspondiente.
```

**starter/main.py:**

```python
# ============================================
# PASO 1: Crear endpoint GET básico
# ============================================
print("--- Paso 1: Endpoint GET básico ---")

# Este endpoint recibe un parámetro de ruta

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samuelduarte-pixel/3171599-Erik-Samuel-Crespo-fast-api](https://github.com/samuelduarte-pixel/3171599-Erik-Samuel-Crespo-fast-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
