---
trigger: always_on
description: Este proyecto es una plataforma SaaS de agendamiento profesional para profesionales de salud, abogados y futuras categorías de servicios profesionales.
---

# AGENTS.md

# Contexto general del proyecto

Este proyecto es una plataforma SaaS de agendamiento profesional para profesionales de salud, abogados y futuras categorías de servicios profesionales.

La plataforma permite que:
- Profesionales se registren, configuren su perfil, disponibilidad, modalidad de atención y calendario.
- Usuarios/clientes/pacientes se registren, busquen profesionales y agenden horas según disponibilidad.
- Administradores gestionen usuarios, profesionales, categorías, especialidades, reservas y configuración general.
- El sistema envíe correos de confirmación y quede preparado para integraciones con Google Meet, Zoom, WhatsApp, pagos e IA.

El objetivo es construir primero un MVP sólido, simple y escalable. No implementar funcionalidades avanzadas antes de tener la base estable.

---

# Stack obligatorio

Backend:
- Python 3.12+
- FastAPI
- SQLAlchemy 2
- Alembic
- PostgreSQL
- Pydantic
- JWT para autenticación
- Passlib o bcrypt para password hashing
- APScheduler para tareas simples iniciales
- SMTP para correos
- Arquitectura preparada para Celery en el futuro, pero no usar Celery en la Fase 1 salvo que se indique explícitamente

Frontend:
- React
- Vite
- TypeScript
- Tailwind CSS
- React Router
- Axios o TanStack Query
- Zustand o Context API para estado global

Infraestructura:
- Docker
- Docker Compose
- Variables de entorno
- `.env.example`
- README con instrucciones claras

Base de datos:
- PostgreSQL
- Migraciones con Alembic
- Seeds iniciales para entorno local

---

# Principios de arquitectura

El proyecto debe priorizar:
- Código limpio
- Separación de responsabilidades
- Modularidad
- Escalabilidad
- Seguridad desde el inicio
- Facilidad de despliegue
- Buenas prácticas DevOps
- Bajo costo operativo inicial

No implementar lógica compleja directamente en endpoints.

La estructura del backend debe separar:
- routers / api
- schemas
- models
- services
- repositories
- core
- db
- integrations
- emails
- utils

La estructura del frontend debe separar:
- pages
- components
- layouts
- api
- features
- routes
- store
- types
- utils

---

# Fases del producto

## Fase 1 - MVP obligatorio

Implementar solamente:

- Autenticación
- Roles: admin, professional, client
- Registro de clientes
- Registro de profesionales
- Perfil profesional
- Categorías
- Especialidades
- Disponibilidad semanal del profesional
- Bloqueo manual de horarios
- Búsqueda de profesionales
- Vista de perfil público del profesional
- Reserva de hora
- Historial de reservas
- Dashboard profesional
- Métricas básicas
- Backoffice administrador mínimo
- Envío de correos
- Meeting provider mock

No implementar todavía:
- Google Meet real
- Zoom real
- WhatsApp real
- Pagos reales
- IA
- Firma digital
- Facturación
- Suscripciones

Esas funcionalidades deben quedar preparadas mediante interfaces o servicios desacoplados.

---

# Roles y permisos

Existen tres roles principales:

## Admin

Puede:
- Ver todo
- Gestionar usuarios
- Gestionar profesionales
- Gestionar categorías
- Gestionar especialidades
- Gestionar reservas
- Ver métricas globales
- Activar/suspender usuarios
- Activar/suspender profesionales
- Modificar configuración general

## Professional

Puede:
- Ver y editar su perfil profesional
- Configurar disponibilidad
- Bloquear fechas
- Ver sus reservas
- Confirmar reservas
- Cancelar reservas
- Completar reservas
- Ver historial de atenciones
- Ver métricas propias
- Agregar notas privadas a una atención

No puede:
- Ver reservas de otros profesionales
- Ver datos administrativos globales
- Modificar categorías o especialidades

## Client

Puede:
- Ver y editar su perfil
- Buscar profesionales
- Ver disponibilidad
- Agendar horas
- Cancelar sus propias reservas futuras
- Ver su historial de reservas

No puede:
- Ver reservas de otros clientes
- Ver notas privadas del profesional
- Acceder al backoffice

---

# Reglas de negocio críticas

- Un profesional no puede tener dos reservas en el mismo horario.
- Un cliente no puede reservar el mismo bloque dos veces.
- La disponibilidad se calcula en base a:
  - reglas semanales del profesional
  - duración de sesión
  - bloqueos manuales
  - reservas existentes
- Las notas privadas del profesional nunca deben ser visibles para el cliente.
- Solo el admin puede ver información global.
- El profesional solo puede ver información asociada a su propio perfil.
- El cliente solo puede ver sus propias reservas.
- Toda acción sensible debe validar permisos por rol.
- Toda reserva debe quedar registrada con historial de cambios.
- No eliminar datos sensibles físicamente si existe historial asociado; preferir soft delete o estados.

---

# Estados de reserva

Usar estos estados:

- pending
- confirmed
- cancelled
- completed
- no_show

No inventar otros estados sin actualizar documentación, schemas, modelos y frontend.

---

# Entidades principales

Mantener como base estas entidades:

- User
- ProfessionalProfile
- ClientProfile
- Category
- Specialty
- ProfessionalSpecialty
- AvailabilityRule
- AvailabilityBlock
- Appointment
- AppointmentHistory
- AuditLog
- SystemSetting

Si se agregan nuevas entidades, justificar brevemente en README o comentario técnico.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fraactal/RealMeet](https://github.com/fraactal/RealMeet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
