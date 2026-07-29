---
trigger: always_on
description: ¿Qué? Archivo de instrucciones para GitHub Copilot y colaboradores del proyecto.
---

# 🎓 Instrucciones del Proyecto — NN Auth System

<!--
  ¿Qué? Archivo de instrucciones para GitHub Copilot y colaboradores del proyecto.
  ¿Para qué? Define TODAS las reglas, convenciones, tecnologías y estándares que se
  deben seguir en cada archivo, commit, test y decisión técnica del proyecto.
  ¿Impacto? Garantiza consistencia, calidad y enfoque pedagógico en todo el código generado.
  Este archivo es la "ley" del proyecto — todo lo que se haga debe alinearse con estas reglas.
-->

---

## 1. Identidad del Proyecto

| Campo               | Valor                                                                                                                   |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Nombre**          | NN Auth System                                                                                                          |
| **Tipo**            | Proyecto educativo — SENA                                                                                               |
| **Propósito**       | Sistema de autenticación completo (registro, login, cambio y recuperación de contraseña) para una empresa genérica "NN" |
| **Enfoque**         | Aprendizaje guiado: cada línea de código y documentación debe enseñar                                                   |
| **Fecha de inicio** | Febrero 2026                                                                                                            |

---

## 2. Stack Tecnológico

### 2.1 Backend (`be/`)

| Tecnología                | Versión | Propósito                                        |
| ------------------------- | ------- | ------------------------------------------------ |
| Python                    | 3.12+   | Lenguaje principal del backend                   |
| FastAPI                   | 0.115+  | Framework web async de alto rendimiento          |
| Uvicorn                   | latest  | Servidor ASGI para ejecutar FastAPI              |
| SQLAlchemy                | 2.0+    | ORM para interactuar con la base de datos        |
| Alembic                   | latest  | Migraciones de base de datos versionadas         |
| Pydantic                  | 2.0+    | Validación de datos y schemas (request/response) |
| pydantic-settings         | latest  | Configuración desde variables de entorno         |
| python-jose[cryptography] | latest  | Creación y verificación de tokens JWT            |
| passlib[bcrypt]           | latest  | Hashing seguro de contraseñas con bcrypt         |
| psycopg2-binary           | latest  | Driver PostgreSQL para Python                    |
| python-multipart          | latest  | Soporte para form data en FastAPI                |
| pytest                    | latest  | Framework de testing                             |
| pytest-asyncio            | latest  | Soporte para tests async con pytest              |
| httpx                     | latest  | Cliente HTTP async para tests de integración     |
| ruff                      | latest  | Linter + formatter ultrarrápido para Python      |

### 2.2 Frontend (`fe/`)

| Tecnología                       | Versión | Propósito                                    |
| -------------------------------- | ------- | -------------------------------------------- |
| Node.js                          | 20 LTS+ | Runtime de JavaScript                        |
| React                            | 18+     | Biblioteca para interfaces de usuario        |
| Vite                             | 6+      | Bundler y dev server ultrarrápido            |
| TypeScript                       | 5.0+    | Superset tipado de JavaScript                |
| TailwindCSS                      | 4+      | Framework CSS utility-first                  |
| React Router                     | 7+      | Enrutamiento del lado del cliente            |
| Axios                            | latest  | Cliente HTTP para comunicación con la API    |
| Vitest                           | latest  | Framework de testing compatible con Vite     |
| Testing Library                  | latest  | Utilidades de testing para componentes React |
| ESLint                           | latest  | Linter para TypeScript/React                 |
| Prettier                         | latest  | Formateador de código                        |
| i18next                          | latest  | Motor de internacionalización (i18n)         |
| react-i18next                    | latest  | Integración de i18next con React (hooks/HOC) |
| i18next-browser-languagedetector | latest  | Detecta idioma del navegador automáticamente |

### 2.3 Base de Datos

| Tecnología     | Versión | Propósito                                       |
| -------------- | ------- | ----------------------------------------------- |
| PostgreSQL     | 17+     | Base de datos relacional principal              |
| Docker Compose | latest  | Orquestación de contenedores (BD en desarrollo) |

### 2.4 Autenticación

| Concepto      | Implementación                                                |
| ------------- | ------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ergrato-dev/proyecto-be_fastapi-fe_react](https://github.com/ergrato-dev/proyecto-be_fastapi-fe_react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
