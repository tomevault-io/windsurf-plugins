---
trigger: always_on
description: >
---


# FastAPI Clean Architecture — Pragmático

Este skill define cómo estructurar proyectos **FastAPI** separando lo que vale la pena separar (lógica de negocio, servicios externos, persistencia testeable) sin caer en arquitectura limpia ortodoxa que duplica entidades, mete mappers en cada capa, exige Unit of Work y termina siendo más ceremonia que valor.

**No es arquitectura limpia purista.** Es FastAPI + casos de uso + inyección de dependencias selectiva + capas solo donde pagan. Suficiente para integraciones complejas, multi-disparador (HTTP + Celery + management command) y tests serios, sin volverse engorroso.

## Cuándo usar este skill

Disparalo cuando el usuario pida **crear, refactorizar o explicar**:

- Un endpoint, router o módulo FastAPI nuevo.
- Un caso de uso (`UseCase`) o flujo de aplicación.
- Un repositorio (SQLAlchemy/SQLModel o PyMongo async).
- Una task Celery 5.4 que llame lógica de negocio.
- Dependencias (`Depends`), `lifespan`, settings, schemas Pydantic.
- Auth con Clerk JWT, API keys SHA-256, o ambos.
- Separar lógica que hoy vive mezclada en routers, signals, tasks o handlers.

También aplica cuando el usuario habla de "arquitectura por capas", "ports & adapters", "desacoplar FastAPI", "testear sin tocar la DB", o muestra código con lógica mezclada en routers.

## Filosofía

Cuatro decisiones que definen este enfoque:

1. **FastAPI es la capa de entrega, no el centro.** Routers, `Depends`, `HTTPException` y `response_model` viven en la API layer; la lógica de negocio NO.
2. **Pydantic vive en la frontera HTTP, no en el dominio.** Los schemas validan requests/responses; las entidades de dominio son `@dataclass` puros (cuando paga separarlos).
3. **Los servicios externos (HTTP, colas, terceros) van detrás de un Protocol** — ahí es donde la abstracción paga, porque es lo que duele en tests. ORM con BD de tests muchas veces puede quedar concreto.
4. **Excepciones de dominio, no HTTPException, dentro de use cases.** El mapeo a HTTP vive en `@app.exception_handler`.

Regla mental para decidir si algo va detrás de Protocol: **"¿En los tests me duele tener la implementación real?"**. Si sí (red, secrets, latencia, errores impredecibles) → Protocol + fake. Si no (ORM con factories y BD de tests) → clase concreta, abstraer cuando duela.

## Cuándo aplicar el patrón completo

Aplicá **Domain + Application + Infrastructure + API** cuando se cumple al menos uno:

- **Integración con un servicio externo** (API REST, webhook saliente, cola).
- La operación se dispara desde **múltiples lugares** (router + Celery + command).
- Hay **reglas de negocio no triviales** que vale testear sin levantar FastAPI ni tocar la red.
- Orquesta **varias fuentes** (DB + HTTP + cache + cola) en una sola operación.

**No apliques el patrón** para CRUD plano. Si tu endpoint es "lista, crea, actualiza un modelo SQLModel sin reglas", un router con `SQLModel` directo + `response_model` resuelve más rápido y más claro. Forzar capas vacías sin valor agrega ruido.

## Modelo de capas

| Capa | Responsabilidad | Qué SÍ tiene | Qué NO tiene |
|------|-----------------|--------------|--------------|
| **Domain** | Entidades, value objects, reglas de dominio, excepciones, ports | Python puro, dataclasses, Protocols | `fastapi`, ORM, `HTTPException`, BSON, `Request` |
| **Application** | Use cases, DTOs (commands/queries), errores de aplicación | Depende de Protocols de domain | Implementaciones concretas, `HTTPException`, status codes |
| **Interface / API** | Routers, schemas Pydantic, deps, exception handlers, auth deps | `APIRouter`, `Depends`, `response_model`, `BackgroundTasks` | Lógica de negocio, queries ORM directas |
| **Infrastructure** | Implementación de repos, clientes HTTP, tasks Celery, mongo_indexes | SQLAlchemy/PyMongo, `requests`, `Celery` | Reglas de negocio |
| **Composition root** | `main.py`, `lifespan`, wiring | Instanciación de clientes, registro de handlers | — |

## Estructura recomendada

Feature-modules: organizar por dominio, no por tipo de archivo. Cada módulo es autónomo.

```
app/
├── main.py                              # FastAPI app + register_exception_handlers + lifespan
├── workers/
│   └── celery_app.py                    # composition root de Celery (separado de FastAPI)
├── core/
│   ├── config.py                        # Settings (pydantic-settings) + get_settings()
│   ├── exceptions.py                    # ApplicationError, PermissionDenied, ValidationError
│   └── security.py                      # helpers crypto (api_key hash, compare_digest)
├── api/
│   ├── deps.py                          # providers: get_db, get_<feature>_repository, get_<use_case>
│   ├── exception_handlers.py            # register_exception_handlers(app)
│   ├── auth/
│   │   ├── clerk_auth.py                # get_user_principal (Clerk JWT)
│   │   └── api_key_auth.py              # get_service_principal (API key SHA-256)
│   └── v1/
│       ├── users.py                     # APIRouter
│       └── projects.py
└── modules/
    └── users/
        ├── domain/
        │   ├── entities.py              # User dataclass
        │   ├── value_objects.py
        │   ├── exceptions.py            # UserNotFound, UserAlreadyExists
        │   └── ports.py                 # UserRepository (Protocol)
        ├── application/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fulit103/fastapi-clean](https://github.com/fulit103/fastapi-clean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
