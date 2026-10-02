---
trigger: always_on
description: La **privacidad de los datos es lo primero, siempre**.
---

# Directrices de Desarrollo de Jarvis

La **privacidad de los datos es lo primero, siempre**.

Toda salida de línea de comandos visible para el usuario debe utilizar emojis. En especial, debe incluirse un emoji inicial para comenzar las líneas que indiquen de qué trata esa línea. La salida debe utilizar espacios de indentación para establecer una jerarquía visual y procurar que sea lo más fácil posible de revisar rápidamente.

**Excepción:** los scripts `.bat` de Windows no pueden utilizar emojis, ya que `cmd.exe` no representa Unicode correctamente.

---

Cualquier punto importante de nuestros flujos lógicos debe tener logs de depuración mediante el método `debug_log` de:

```text
src/jarvis/debug.py
```

Evita registrar información excesiva para mantener los logs fáciles de leer y útiles para actuar sobre ellos.

---

## Cambios de código y archivos de especificación

Cualquier cambio de código debe cumplir perfectamente con nuestros archivos de especificación. Si no es posible hacerlo, debes solicitar al usuario que confirme los cambios. Esa confirmación también debe propagarse a los propios archivos de especificación.

Los archivos de especificación utilizan el formato:

```text
*.spec.md
```

y se encuentran junto al código que implementan.

**Siempre busca los archivos de especificación relacionados antes de comenzar cualquier trabajo.**

Cuando se corrija cómo debería funcionar algo, comprueba si existe una especificación para ese comportamiento y determina si necesita actualizarse.

---

# Registro de Archivos de Especificación

| Archivo de especificación                             | Cubre                                                                                                                                                     | Principios clave                                                                                                                                                                                             |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `src/desktop_app/desktop_app.spec.md`                 | Aplicación de bandeja del sistema, flujo de inicio, integración con daemon, ventanas, tema y actualizaciones                                              | Desktop está separado del núcleo; jarvis no tiene conocimiento de `desktop_app`                                                                                                                              |
| `src/desktop_app/settings_window.spec.md`             | UI de configuración generada automáticamente a partir de metadatos de configuración                                                                       | Basada en metadatos; solo se escriben valores diferentes a los predeterminados; conserva claves desconocidas                                                                                                 |
| `src/desktop_app/setup_wizard.spec.md`                | Asistente de primera ejecución, Ollama, modelos, Whisper y ubicación                                                                                      | Fricción mínima; solo aparece cuando requiere una acción del usuario; no configura todo                                                                                                                      |
| `src/jarvis/dictation/dictation.spec.md`              | Motor de dictado mediante pulsación prolongada, hotkey y pegado desde el portapapeles                                                                     | Independiente del pipeline del asistente; comparte el modelo Whisper; utiliza una bandera de pausa en el listener                                                                                            |
| `src/jarvis/listening/listening.spec.md`              | Listener de voz, detección de palabra de activación y pipeline de audio                                                                                   | -                                                                                                                                                                                                            |
| `src/jarvis/reply/reply.spec.md`                      | Generación de respuestas LLM, uso de herramientas y perfiles                                                                                              | Las herramientas devuelven datos sin procesar; los perfiles gestionan el formato                                                                                                                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isairey/AsistenteIAPrivadoJARVIS](https://github.com/isairey/AsistenteIAPrivadoJARVIS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
