---
trigger: always_on
description: Este es el paquete QuinotoSpec — una metodología y sistema de configuración de agentes para desarrollo asistido por IA. Los agentes operan en proyectos que usan QuinotoSpec siguiendo el flujo de trabajo "Proposal First" / "Context Slicing".
---

# Guía para Agentes QuinotoSpec

Este es el paquete QuinotoSpec — una metodología y sistema de configuración de agentes para desarrollo asistido por IA. Los agentes operan en proyectos que usan QuinotoSpec siguiendo el flujo de trabajo "Proposal First" / "Context Slicing".

---

## 1. Comandos de Build, Lint y Test

Dado que este es un **paquete de configuración de agentes** (no una aplicación en tiempo de ejecución), los comandos tradicionales de build/test no aplican. Sin embargo, al trabajar en **proyectos objetivo** que usan QuinotoSpec:

### Comandos de Análisis de Proyecto
```bash
# Detectar stack del proyecto
/quinotospec-stack-detect

# Ejecutar discovery completo
/quinotospec.discovery

# Validar estado del sistema antes de workflows críticos
/quinotospec-validate --full
```

### Validación de Sintaxis
```bash
# Validar todos los archivos QuinotoSpec
/quinotospec-syntax-validate --type all

# Validar una propuesta específica
/quinotospec-syntax-validate --type proposal --slug <slug-name> --strict
```

### Ejecutar Tests en Proyectos Objetivo
```bash
# Leer el perfil de stack para encontrar el comando de test correcto
cat .quinoto-spec/discovery/01-stack-profile.md

# Patrones comunes: npm test, pytest, bundle exec rspec, go test ./..., cargo test
```

**Patrones para un solo test**: `npm test -- --testPathPattern=filename`, `pytest -k test_name`, `bundle exec rspec spec/path/to/spec.rb`, `go test -run TestName ./...`

---

## 2. Convenciones de Código

### Principios Fundamentales
- **Proposal First**: Siempre crear una propuesta técnica antes de escribir código
- **Context Slicing**: Discovery → Propuesta → User Stories → Tareas
- **Nunca editar archivos de especificación manualmente**: Usar skills como `quinotospec-update-changelog`
- **Toda edición del agente debe registrarse**: Después de cualquier cambio significativo, ejecutar `/quinotospec-update-changelog` para documentar la acción en el historial

### Convenciones de Nombrado de Archivos
- **Workflows**: `quinotospec.<workflow-name>.md`
- **Skills**: `quinotospec-<skill-name>/SKILL.md`
- **Rules**: `quinotospec-rules.md`
- **Scripts Temporales**: `.quinoto-spec/scripts/temp_*.{py,sh,js}` (NUNCA en raíz del proyecto)

### Convenciones de Nombrado de Branches (CRÍTICO)
```
feature/{{TASK_ID}}-descripcion-en-kebab-case
bugfix/{{TASK_ID}}-descripcion-en-kebab-case
```
Ejemplos: `feature/TSK-AUTH-001-add-login-endpoint`, `bugfix/US-ABC-123-fix-validation`

### Convención de Archivado
- Usar carpeta `_archived/` (NUNCA prefijo `__`)
- Estructura: `.quinoto-spec/proposals/{SLUG}/_archived/` para archivos

---

## 3. Flujo de Trabajo QuinotoSpec

```
1. Discovery (/quinotospec.discovery)           → Entender contexto del proyecto
2. Propuesta (/quinotospec.create-proposal)     → Definir solución técnica
3. User Stories (/quinotospec.create-user-stories) → Desglosar en stories de valor
4. Tareas (/quinotospec.create-tasks)           → Convertir en tareas atómicas
5. Aplicar (/quinotospec.apply)                → Implementar tarea
6. Marcar Completado (/quinotospec-mark-done)   → Actualizar specs y archivar
```

### Regla del Registro de Prefijos
- Cada propuesta necesita un prefijo único que combine un mnemónico de 4 letras + un sufijo de 4 caracteres (ej. `AUTH-a1b2`)
- Registrar en `.quinoto-spec/prefix-registry.md`
- NUNCA inventar prefijos que no estén en el registro o que no sigan el formato de idempotencia

### Verificación de Acuerdo de Producto (BLOQUEANTE)
Antes de `/quinotospec.create-proposal`:
1. Verificar `.quinoto-spec/discovery/08-product-and-agreements.md`
2. Si solo tiene placeholders o está vacío → DETENER y notificar al usuario para completar DoR/DoD

---

## 4. Manejo de Errores

- **Archivo de tarea faltante** → notificar al usuario y detener
- **ID de tarea no encontrado** → reportar error con sugerencias
- **Interrupciones de workflow** → guardar `ultimo_paso_completado` en YAML para reanudar
- **Fallos de validación** → usar `/quinotospec-validate --strict` para bloquear

---

## 5. Ubicaciones de Archivos QuinotoSpec

Una vez ejecutado el `/quinotospec.discovery`, los 8 archivos en `.quinoto-spec/discovery/` documentan exhaustivamente la naturaleza del proyecto: stack tecnológico, arquitectura, endpoints, modelos de datos, prácticas DevOps, deuda técnica y acuerdos de producto. Estos archivos constituyen la fuente autoritativa de contexto para cualquierAgent antes de proponer o implementar cambios.

```
.quinoto-spec/
├── discovery/                    # 8 archivos de especificación del proyecto
│   ├── 01-stack-profile.md       # Stack tecnológico, frameworks, test runners
│   ├── 02-overview.md            # Resumen ejecutivo, estructura de carpetas
│   ├── 03-architecture.md       # Diagrama de arquitectura, patrones de diseño
│   ├── 04-endpoints-and-openapi.md  # Endpoints REST/GraphQL, OpenAPI spec
│   ├── 05-data-and-services.md   # Modelos de datos, servicios externos
│   ├── 06-devops-ci-security.md  # CI/CD, auditoría de dependencias, seguridad
│   ├── 07-findings-and-recommendations.md  # Deuda técnica, vulnerabilidades

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Quinoto-Tech/QuinotoSpec](https://github.com/Quinoto-Tech/QuinotoSpec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
