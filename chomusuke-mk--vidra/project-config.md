---
trigger: always_on
description: ¡Bienvenido! Eres un agente de inteligencia artificial (IA) o asistente de código encargado de colaborar en el proyecto **Vidra**. Esta guía contiene las instrucciones, estándares y arquitectura del proyecto para que puedas empezar a trabajar de manera rápida y eficiente.
---

# 🤖 Guía para Agentes de IA en Vidra

¡Bienvenido! Eres un agente de inteligencia artificial (IA) o asistente de código encargado de colaborar en el proyecto **Vidra**. Esta guía contiene las instrucciones, estándares y arquitectura del proyecto para que puedas empezar a trabajar de manera rápida y eficiente.

## 🎯 Objetivo General del Proyecto

**Vidra** es un gestor de descargas de video avanzado, multiplataforma (Android, Windows, Linux, macOS), construido con:

- **Frontend (Cliente):** Flutter (Dart) con enfoque en un diseño moderno, responsivo y usando principios de _Clean Architecture_.
- **Backend (Motor de descarga):** Python empaquetado como un proceso aislado (Isolate) usando `serious_python`. Utiliza internamente `yt-dlp` y `FFmpeg`/`QuickJS`.
- **Comunicación:** A través de una API REST HTTP local en un puerto dinámico y seguro.

---

## 📂 Estructura del Proyecto

El código está fuertemente separado entre el frontend (UI) y el backend (lógica de descarga). A continuación, se detallan los directorios clave:

- `/lib`: Contiene el código fuente en Dart/Flutter. Mantiene la separación de preocupaciones (UI, lógica de negocio/estado con `Provider`, servicios HTTP, manejo de archivos y modelos).
- `/app`: Contiene el código fuente del backend en Python y sus dependencias (`requirements/`).
- `/docs`: Documentación técnica detallada. **DEBES** consultar estos archivos cuando trabajes en características complejas.
  - `docs/system-architecture.md`: Detalles de cómo se comunica Flutter con el backend de Python.
  - `docs/client-flows.md`: Flujos principales de la UI y ciclo de vida de la app.
  - `docs/development-guide.md`: Guía de desarrollo, testing y _troubleshooting_.
- `/android`, `/windows`, `/linux`, `/macos`: Configuraciones nativas de las plataformas soportadas.
- `/.github/workflows`: Definición de los pipelines de CI/CD (GitHub Actions) para el empaquetado de dependencias nativas (como FFmpeg/QuickJS) y la automatización de _releases_.
- `/test` e `/integration_test`: Pruebas unitarias, de widgets y de integración en Dart.

---

## 📖 Orden de Lectura Sugerido

Antes de implementar modificaciones significativas, asegúrate de comprender la base de conocimiento del proyecto. Como agente, **debes priorizar leer estos archivos** utilizando tus herramientas de lectura:

1. **`README.md`**: Para entender cómo funciona el proyecto a alto nivel, cómo se compila y el rol de dependencias críticas como `serious_python`.
2. **`pubspec.yaml`**: Para conocer las dependencias de Dart disponibles en el proyecto (ej. `provider`, `dio`/`http`, `flutter_localizations`). ¡Usa lo que ya está instalado!
3. **`docs/system-architecture.md`** (Si vas a tocar la interacción entre Dart y Python): Para entender los puertos de la API local, encriptado de datos y cómo se sincroniza el progreso.
4. **`docs/development-guide.md`** (Si necesitas probar o compilar algo complejo): Para entender cómo manejar binarios nativos (FFmpeg, QuickJS) durante el entorno de desarrollo.

---

## 🛠️ Estándares y Convenciones de Código

### Frontend: Flutter / Dart (`/lib`)

- **Gestión de Estado:** Se usa el paquete `provider`. Respeta el flujo unidireccional. La UI consume _Providers_ y no almacena estado complejo de negocio internamente.
- **Internacionalización (i18n):** Cualquier texto visible en la interfaz debe estar localizado usando los archivos `i18n/*.jsonc` y el modelo `AppStringKey` en `lib/features/locales/domain/locale.dart`. **Cero _strings hardcodeados_** en las vistas.
  - **Convención de prefijos i18n:**
    - `s_` y `s_*_desc`: Pantalla de configuración y sus descripciones informativas.
    - `d_`: Pantalla principal de descargas.
    - `dd_`: Detalles de descarga.
    - `sw_`: Modal y flujo de selección de elementos (Selection Wrapper).
    - `shw_`: Envoltura de enlace compartido (Share Wrapper).
    - `ov_`: Pantalla y modal flotante de Overlay.
    - `p_`: Pantalla de permisos.
    - `sd_`: Estado y detalles del sistema.
    - `tu_`: Textos de tutoriales (`TutorialUtils`).
    - `dc_`: Tarjeta de descarga (`DownloadCard`).
    - `fe_`: Diálogo de error fatal.
  - Al agregar un string nuevo, añádelo en `i18n/en.jsonc`, genera su getter en `AppStringKey` y regístralo en `_allAppStrings`.
- **Pantalla de Configuración (`SettingsScreen`):**
  - Todas las opciones se definen dentro de `_getAllSettings` mediante `_SettingDef`.
  - Emplea exclusivamente los componentes optimizados de `lib/shared/widgets/`: `LazyDropdown`, `LazyTextField`, `LazyList`, `LazyMap` y `SettingRow`.
  - Modificaciones en las opciones de descarga deben reflejarse en `DownloadOptions` (`toJson`/`fromJson`) y en `_applyDynamicDefaults` de `SettingsController`.
- **Overlay de Compartir (`QuickShareOverlay`):**
  - Se ejecuta en un Isolate secundario e independiente (`overlayMain` con `@pragma("vm:entry-point")`).
  - La comunicación para encolar descargas desde el overlay hacia el motor se realiza mediante IPC con `IsolateNameServer.lookupPortByName('vidra_backend_port')`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chomusuke-mk/vidra](https://github.com/chomusuke-mk/vidra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
