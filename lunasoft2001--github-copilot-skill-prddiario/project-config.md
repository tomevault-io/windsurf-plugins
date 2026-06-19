---
trigger: always_on
description: Gestiona tareas diarias con formato jerárquico legible, documentación completa (descripción + solución) con timestamps, y genera reportes automáticos de horas trabajadas. Usa cuando necesites crear PRD diario, registrar tareas completadas, gestionar pendientes, o generar reportes de horas. Incluye scripts Python/PowerShell para automatizar todo.
---


# PRD Diario - Gestor de Tareas Diarias

## Descripción General

Automatiza la creación y organización de Documentos de Requisitos de Productos (PRD) diarios para un seguimiento estructurado de tareas. **Mantiene separadas las carpetas de PRD, trabajo diario, reportes y archivos**. Ideal para:

- **Organización flexible** - Configura dónde guardar cada tipo de documento
- **Carpetas PRD centralizadas** - Todos los `PRD_YYYYMMDD.md` en un solo lugar
- **Trabajo diario organizado** - Carpetas YYMMDD separadas para documentos del día
- **Reportes consolidados** - Resúmenes y análisis en una carpeta dedicada
- **Archivo histórico** - Días completados archivados para referencia
- **Registro de tareas** con timestamps exactos y documentación completa
- **Resúmenes automáticos** que analizan metadatos de archivos
- **Rastreo de inicio de jornada** usando fecha de creación de documentos
- **Reportes de horas trabajadas** y auditoría completa

## Cuándo Usar Este Skill

Use este skill cuando:

- Necesite **iniciar el día** y crear la carpeta + PRD diario
- Quiera **registrar tareas** completadas con hora exacta  
- Deba **documentar soluciones** de manera estructurada
- Tenga **tareas pendientes** que requieran seguimiento
- Necesite **guardar conversaciones** o documentos del día en un solo lugar
- Quiera **generar resumen del día** automático con metadatos
- Necesite **reportes de horas** trabajadas al final del día
- Desee saber **cuándo empezó el día** (primer documento creado)
- El usuario pida "crear PRD diario", "iniciar día", "registrar tarea", "resumen del día" o "generar reporte de horas"

## Flujo de Trabajo

### Fase 0: Configuración Inicial (IMPORTANTE!)

**Primera ejecución - Configura tus carpetas:**

```bash
python scripts/setup_config.py
```

Este wizard interactivo te permite elegir dónde guardar:
- 📄 **PRD_DOCUMENTS**: Donde van todos los `PRD_YYYYMMDD.md`
- 📁 **DAILY_WORK**: Donde se crean carpetas YYMMDD por día
- 📊 **REPORTS**: Donde se generan resúmenes y reportes
- 📦 **ARCHIVES**: Donde archivar días completados (opcional)

**Estructura recomendada:**
```
~/Documents/prd_diarios/
  ├── PRD_DOCUMENTS/              ← Todos los PRD centralizados
  │   ├── PRD_20260225.md
  │   ├── PRD_20260226.md
  │   └── PRD_20260227.md
  ├── DAILY_WORK/                 ← Trabajo diario por carpetas
  │   ├── 260225/
  │   │   ├── README.md
  │   │   ├── conversacion_cliente.md
  │   │   └── notas_meeting.md
  │   ├── 260226/
  │   └── 260227/
  ├── REPORTS/                    ← Resúmenes y reportes
  │   ├── RESUMEN_260225.md
  │   ├── RESUMEN_260226.md
  │   ├── HORAS_PRD_20260225.md
  │   └── HORAS_PRD_20260226.md
  └── ARCHIVES/                   ← Días completados
      └── (semanas antiguas)
```

**Ver configuración actual:**
```bash
python scripts/setup_config.py --show
```

**Resetear configuración:**
```bash
python scripts/setup_config.py --reset
```

### Fase 1: Inicio del Día (NUEVO)

Mientras trabajas:

1. **Registrar tareas**: Añade tareas completadas con timestamps
2. **Guardar documentos**: Guarda conversaciones, notas, archivos en la carpeta del día
3. **Actualizar PRD**: Documenta descripción + solución de cada tarea

**Ejemplo de estructura durante el día:**
```
260225/
  ├── README.md
  ├── PRD_20260225.md
  ├── conversacion_cliente_proyecto_X.md
  ├── notas_meeting_equipo.md
  └── diagrama_arquitectura.png
```

Mientras trabajas:

1. **Registrar tareas**: Añade tareas completadas con timestamps en el PRD
2. **Guardar documentos**: Guarda conversaciones, notas, archivos en DAILY_WORK/260225/
3. **Actualizar PRD**: Documenta descripción + solución de cada tarea en PRD_DOCUMENTS/

**Estructura típica durante el día:**
```
DAILY_WORK/260225/
  ├── README.md
  ├── conversacion_cliente_proyecto_X.md
  ├── notas_meeting_equipo.md
  └── diagrama_arquitectura.png

PRD_DOCUMENTS/
  └── PRD_20260225.md          ← Actualizado con tareas
```

### Fase 3: Fin del Día

Al terminar la jornada:

1. **Generar resumen del día**: Ejecuta `generate_day_summary.py`
2. **Analiza metadatos**: Lee fechas de creación de todos los archivos
3. **Calcula horas**: Determina inicio (primer archivo) y tareas realizadas
4. **Crea reporte**: Genera RESUMEN_YYMMDD.md con toda la información

**Estructura final:**
```
260225/
  ├── README.md
  ├── PRD_20260225.md
  ├── conversacion_cliente_proyecto_X.md
  ├── notas_meeting_equipo.md
  ├── diagrama_arquitectura.png
  ├── RESUMEN_260225.md          # ← Generado automáticamente
  └── HORAS_PRD_20260225.md      # ← Opcional: reporte de horas detallado
```

### Fase 3 (Antigua): Registrar Tareas Realizadas

Para cada tarea completada:

```markdown
### ✅ N. Nombre de la Tarea — **HH:MM**

**Descripción**  
Contexto y motivo de la tarea. Qué problema se resolvía, de dónde venía la solicitud.

**Solución**  
Qué se hizo y cómo se resolvió. Pasos tomados, tecnologías usadas, resultado final.
```

**Ejemplo:**

```markdown
### ✅ 1. Revisar tareas asignadas en Trello — **09:00**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lunasoft2001/github-copilot-skill-prddiario](https://github.com/lunasoft2001/github-copilot-skill-prddiario) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
