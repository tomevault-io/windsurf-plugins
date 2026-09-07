---
trigger: always_on
description: Documento de gobernanza técnica. Todo agente que interactúe con este repositorio
---

# AGENTS.md — G_ACADEMICA

Documento de gobernanza técnica. Todo agente que interactúe con este repositorio
debe leerlo y seguirlo antes de proponer o implementar cambios.

---

## 1. Identidad del proyecto

| Atributo | Valor |
|---|---|
| **Nombre** | G_ACADEMICA |
| **Dominio** | Gestión académica (estudiantes, docentes, cursos, calificaciones, inscripciones, planes de estudio) |
| **Usuarios objetivo** | Administradores académicos, docentes, estudiantes, personal de secretaría |
| **Alcance** | Plataforma para administrar el ciclo de vida académico: inscripción de estudiantes, asignación de docentes, creación y gestión de cursos, registro de calificaciones, generación de reportes, control de planes de estudio |
| **Estado actual** | Proyecto greenfield — sin código, sin dependencias, sin historia. Cada decisión fundacional debe documentarse como ADR. |

El agente debe entender que **todas las decisiones técnicas deben servir al dominio académico**. No se introducen tecnologías o patrones por moda: cada elección debe justificarse frente a un problema real del negocio.

---

## 2. Stack tecnológico

> Este stack es **propuesto por defecto** hasta que existan archivos de configuración
> que lo contradigan. El agente debe leer `package.json`, `tsconfig*`, `composer.json`,
> `Cargo.toml` o equivalentes antes de asumir el stack.

| Tecnología | Rol | Motivación |
|---|---|---|
| **Node.js + TypeScript** | Runtime y lenguaje | Tipado estático para reducir errores en dominio complejo; ecosistema maduro para APIs; las interfaces y tipos de TS permiten aplicar DIP e ISP con naturalidad |
| **Express / Fastify** | Framework HTTP | Ligero, extensible con plugins, sin opiniones arquitectónicas fuertes que choquen con Clean Architecture |
| **PostgreSQL** | Base de datos relacional | El dominio académico exige integridad referencial (estudiantes ↔ inscripciones ↔ cursos ↔ calificaciones); las relaciones many-to-many y las restricciones CHECK son esenciales |
| **Prisma / Drizzle** | ORM / query builder | Tipado seguro desde la BD hasta la aplicación; migrations como código; evita el anemic domain model al separar persistencia de dominio |
| **Vitest** | Testing | Rápido, compatible con TypeScript nativo, soporte para cobertura y mocks |
| **Docker** | Entorno de desarrollo y despliegue | Reproducibilidad; evita el "en mi máquina funciona"; permite levantar PostgreSQL sin instalación manual |

Cualquier cambio en el stack debe documentarse como ADR en `/docs/decisions/`.

---

## 3. Arquitectura del sistema

Se adopta **Clean Architecture** (también conocida como arquitectura hexagonal o de puertos y adaptadores) por las siguientes razones del dominio:

- El negocio académico tiene reglas complejas (cálculo de promedios, prerequisitos, choques de horario) que **no deben depender de frameworks ni de la base de datos**.
- Los casos de uso deben ser testeables sin infraestructura.
- La UI puede cambiar (web, mobile, API REST, GraphQL) sin afectar el núcleo.

### Diagrama de capas

```
┌─────────────────────────────────────────────────────────┐
│                    Infraestructura                        │
│  (Express, Prisma, JWT, Nodemailer, AWS S3, logger...)  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Presentation / API                   │   │
│  │  (controllers, middleware, serializers, routes)   │   │
│  └──────────────────┬───────────────────────────────┘   │
│                     │                                    │
│  ┌──────────────────▼───────────────────────────────┐   │
│  │              Application / Use Cases              │   │
│  │  (orquestar flujos, coordinar repos y servicios)  │   │
│  └──────────────────┬───────────────────────────────┘   │
│                     │                                    │
│  ┌──────────────────▼───────────────────────────────┐   │
│  │                   Domain                           │   │
│  │  (entidades, value objects, domain services,      │   │
│  │   repository interfaces, domain events)            │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Reglas de dependencia

- **Domain**: NO depende de ninguna otra capa. Es el centro del sistema.
- **Application**: depende de Domain. NO depende de infraestructura.
- **Infrastructure**: depende de Domain y Application. Implementa interfaces definidas en Domain.
- **Presentation**: depende de Application. Conecta el mundo exterior con los casos de uso.

La lógica de negocio vive exclusivamente en **Domain** (entidades, value objects, domain services) y **Application** (casos de uso). Nunca en infraestructura o presentación.

### Estructura de carpetas esperada

```
src/
├── domain/
│   ├── entities/
│   ├── value-objects/
│   ├── services/
│   ├── repositories/  (interfaces)
│   └── events/
├── application/
│   ├── use-cases/
│   ├── dtos/
│   └── ports/         (interfaces de salida)
├── infrastructure/
│   ├── persistence/   (implementaciones de repositorios)
│   ├── auth/
│   ├── email/
│   ├── logging/
│   └── config/
├── presentation/
│   ├── controllers/
│   ├── middleware/
│   ├── serializers/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cj200xyz/Gestion_Academica](https://github.com/cj200xyz/Gestion_Academica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
