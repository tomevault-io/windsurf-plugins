---
trigger: always_on
description: > **Note**: This file contains research context from CodeViz. Most recent contexts are at the bottom.
---

# CodeViz Research Context

> **Note**: This file contains research context from CodeViz. Most recent contexts are at the bottom.

---

## Research Query

muestrame el flujo del codigo de este proyecto

*Session: cbaa56dfc59ba3a0cb5ce17d841cd999 | Generated: 03/07/2025, 11:14:22*

### Analysis Summary

El proyecto "asim-solar-project" es el desarrollo de un plugin de WordPress llamado "solar-project" y otro plugin relacionado con Gravity Forms, "coco-gravity-form-map-field".

Aquí te presento un flujo de código de alto nivel basado en la estructura de directorios:

# Flujo de Código del Proyecto asim-solar-project

Este proyecto es una aplicación de WordPress que integra funcionalidades personalizadas a través de plugins. El flujo principal se centra en la interacción del usuario con formularios de Gravity Forms y la lógica de negocio relacionada con el proyecto solar.

## 1. Inicialización de WordPress y Plugins

Como todo plugin de WordPress, el flujo comienza con la carga de WordPress, que a su vez inicializa los plugins instalados.

*   **WordPress Core**:
*   **Carga de Plugins**: Durante la inicialización de WordPress, los plugins en el directorio `plugins/` son cargados.
    *   `plugins/solar-project/solar-project.php`: Archivo principal del plugin "solar-project". Este archivo probablemente contiene los hooks de WordPress para inicializar la funcionalidad del plugin.
    *   `plugins/coco-gravity-form-map-field/coco-gravity-form-map-field.php`: Archivo principal del plugin de Gravity Forms, que extiende la funcionalidad de los formularios con campos de mapa.
	 *   `plugins/gravityforms/gravityforms.php` es un plugin conocido de WordPress, y necesario para que estos plugins funcionen.

## 2. Plugin "solar-project"

Este plugin es el corazón de la lógica de negocio del proyecto solar, dependiente de `gravityforms` y de `coco-gravity-form-map-field`

### 2.1. Configuración y Utilidades

*   `plugins/solar-project/inc/class-enqueue.php`: Probablemente maneja la encolación de scripts y estilos CSS/JS necesarios para el plugin.
*   `plugins/solar-project/inc/class-helper.php`: Contiene funciones de utilidad y ayuda para diversas operaciones dentro del plugin.
*   `plugins/solar-project/inc/class-notifications.php`: Gestiona las notificaciones, posiblemente relacionadas con el progreso del proyecto solar o interacciones del usuario.
*   `plugins/solar-project/assets/notifications/`: Contiene plantillas HTML para diferentes tipos de notificaciones.

### 2.2. Integración con Gravity Forms

*   `plugins/solar-project/inc/class-gravity-hooks.php`: Este archivo es crucial, ya que probablemente contiene los hooks y filtros de WordPress que interactúan con Gravity Forms. Aquí es donde se procesan los datos enviados a través de los formularios.
*   `plugins/solar-project/inc/class-gravity-model-panel.php`: Podría estar relacionado con la visualización de datos o modelos dentro de Gravity Forms, posiblemente para la configuración de paneles solares.
*   `plugins/solar-project/inc/class-model-panel.php`: Complementa la funcionalidad de `class-gravity-model-panel.php`, definiendo la lógica para los modelos de paneles.
*   `plugins/solar-project/inc/render-step-4.php`: Sugiere un flujo de múltiples pasos, donde este archivo se encarga de renderizar la interfaz o procesar la lógica del "paso 4".

### 2.3. Lógica de la Interfaz de Usuario (Frontend)

Los archivos JavaScript en `plugins/solar-project/src/` son responsables de la interactividad en el lado del cliente.

*   `plugins/solar-project/src/index.ts`: El archivo principal de TypeScript que orquesta la lógica del frontend.
*   `plugins/solar-project/src/step1_functions.ts`, `step2_functions.ts`, `step3_functions.ts`, `step4_functions.ts`: Indican un flujo de usuario paso a paso, donde cada archivo maneja la lógica específica de un paso. Esto podría ser para un configurador de paneles solares o un proceso de cotización.
*   `plugins/solar-project/src/drawing-helpers.ts`, `trigonometry-helpers.ts`: Sugieren funcionalidades de dibujo y cálculos trigonométricos, lo que es coherente con un proyecto solar que podría implicar el diseño de instalaciones.
*   `plugins/solar-project/src/setup-solar-panels.ts`: Probablemente maneja la configuración y visualización de los paneles solares en la interfaz.
*   `plugins/solar-project/src/notification-api.ts`: Lógica del lado del cliente para interactuar con las notificaciones.

### 2.4. Integración con APIs Externas

*   `plugins/solar-project/inc/api/class-API.php`: Clase base para interactuar con APIs externas.
*   `plugins/solar-project/inc/api/class-google-maps-api.php`: Maneja la integración con la API de Google Maps, lo cual es fundamental para un proyecto solar que requiere información de ubicación y mapeo.
*   `plugins/solar-project/inc/api/class-solar-api.php`: Podría ser una API personalizada para obtener datos solares o interactuar con servicios específicos del sector solar.
*   `plugins/solar-project/assets/response-google-maps.json`, `response-solar-findbuilding.json`: Ejemplos de respuestas de APIs, lo que indica que el plugin consume datos de servicios externos.

## 3. Plugin "coco-gravity-form-map-field"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cobianzo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
