---
trigger: always_on
description: Este archivo describe las convenciones, patrones y restricciones del proyecto para que cualquier agente de IA pueda contribuir correctamente sin romper la arquitectura existente.
---

# AGENTS.md — Guía para agentes de IA

Este archivo describe las convenciones, patrones y restricciones del proyecto para que cualquier agente de IA pueda contribuir correctamente sin romper la arquitectura existente.

---

## Qué es este proyecto

Backend Spring Boot para el sistema de gestión del Club CIPOLFLO. Expone una API REST que maneja clientes (particulares y socios), servicios ofrecidos, reservas y finanzas (ingresos/egresos).

- **Lenguaje:** Java 21
- **Framework:** Spring Boot 3.x
- **Build:** Gradle con Kotlin DSL (`build.gradle.kts`)
- **Base de datos:** PostgreSQL
- **Puerto:** 8080

---

## Estructura del proyecto

```
src/main/java/com/cipolflo/server/
├── shared/                     ← Clases transversales. No depende de ningún módulo.
├── clientes/                   ← Módulo de clientes y socios
├── servicios/                  ← Módulo de servicios del club
├── reservas/                   ← Módulo de reservas
├── finanzas/                   ← Módulo de ingresos y egresos
└── integraciones/              ← Integración con servicios externos (no es un dominio de negocio)
```

Cada módulo de dominio (`clientes`, `servicios`, `reservas`, `finanzas`) tiene esta
estructura interna fija:

```
{modulo}/
├── domain/
│   └── enums/
├── repository/
├── service/
├── dto/
└── controller/
```

No crear carpetas fuera de este esquema salvo acuerdo explícito.

### `integraciones/` — excepción acordada al esquema de módulo

`integraciones/` no sigue la estructura `domain/repository/service/dto/controller` de los
módulos de negocio: es la capa de integración con servicios externos (Telegram, y a
futuro otros canales de mensajería o IA). Se parte en **núcleo agnóstico + adaptador**:

```
integraciones/
├── mensajeria/      ← núcleo: no sabe qué es Telegram. Solo conoce sus propios
│                       puertos (CanalMensajeria, RegistroDestinatarios) y tipos
│                       genéricos (destinatarioId: String). Acá viven las tools de IA,
│                       el asistente (Spring AI) y la memoria conversacional.
└── telegram/        ← adaptador: conoce el formato de update de Telegram, implementa
                        los puertos de mensajeria/, expone el webhook.
```

Regla: si se agrega un canal nuevo (ej. WhatsApp), se escribe un adaptador nuevo
(`integraciones/whatsapp/`) implementando los mismos puertos de `mensajeria/` — el núcleo
no cambia. Ver [`docs/telegram-bot-arquitectura.md`](docs/telegram-bot-arquitectura.md)
para el detalle completo.

---

## Convenciones de código

### Lombok

Todas las entidades usan Lombok. El patrón estándar en entidades es:

```java
@Getter
@Setter
@NoArgsConstructor
```

- Nunca escribir getters/setters a mano si Lombok los puede generar.
- Cuando un campo no debe ser modificable desde fuera de la entidad, usar `@Setter(AccessLevel.NONE)` en ese campo específico (ver `Reserva.java`).
- Cuando el constructor público no debe existir (entidad con factory method), usar `@NoArgsConstructor(access = AccessLevel.PROTECTED)`.

### Entidades JPA

- Todas las entidades extienden `AuditableEntity` (provee `createdAt` y `updatedAt` automáticos vía JPA Auditing).
- Los enums se persisten siempre como `@Enumerated(EnumType.STRING)`, nunca `ORDINAL`.
- Las IDs son `Long` con `@GeneratedValue(strategy = GenerationType.IDENTITY)`.
- Los campos requeridos llevan `@Column(nullable = false)`.
- Los campos únicos llevan `@Column(unique = true, nullable = false)`.

### Herencia de entidades

Hay dos jerarquías con `SINGLE_TABLE`:

| Tabla     | Discriminador | Subtipos              |
| --------- | ------------- | --------------------- |
| `cliente` | `tipo`        | `PARTICULAR`, `SOCIO` |
| `finanza` | `tipo`        | `INGRESO`, `EGRESO`   |

Si se agrega un nuevo subtipo, debe anotarse con `@DiscriminatorValue("NOMBRE")` y extender la clase abstracta correspondiente. No crear una tabla nueva salvo que la herencia cambie de estrategia.

### Enums compartidos vs de módulo

- Enums usados por **más de un módulo** van en `shared/enums/` (ej. `FormaPago`, `Procedencia`).
- Enums usados por **un solo módulo** van en `{modulo}/domain/enums/` (ej. `EstadoReserva`, `EstadoSocio`).

### Nombres

- Clases en español siguiendo el dominio del negocio (igual que el código existente).
- Interfaces de servicio: `I{Nombre}Service`.
- Implementaciones: `{Nombre}Service`.
- DTOs: `{Nombre}RequestDto` y `{Nombre}ResponseDto`.
- Tests: `{Nombre}Test` para dominio, `{Nombre}ServiceTest` para servicios.

---

## Patrones de dominio — no romper

### Factory method en `Reserva`

`Reserva` tiene constructor protegido. La única forma válida de crear una reserva es:

```java
Reserva r = Reserva.crear(clienteId, servicioId, procedencia, fechaEntrada, fechaSalida, requiereDocumentacion);
```

No agregar un constructor público ni cambiar la visibilidad del existente.

### Máquina de estados de `Reserva`

Las transiciones de estado son:

```
PENDIENTE → CONFIRMADA → EN_CURSO → FINALIZADA
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CIPOLFLO/cipolflo-server](https://github.com/CIPOLFLO/cipolflo-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
