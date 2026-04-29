---
trigger: always_on
description: Esta es una guía de referencia rápida para las herramientas de automatización del navegador disponibles.
---

# Especificaciones de Herramientas del Navegador

Esta es una guía de referencia rápida para las herramientas de automatización del navegador disponibles.

## Detalles de las Herramientas

### Tool: `browser_click`
Realiza un clic en una página web.

**Parámetros:**
| Nombre    | Tipo   | Descripción                                                                                             |
|-----------|--------|---------------------------------------------------------------------------------------------------------|
| `element` | string | Descripción legible por humanos del elemento para obtener permiso para interactuar con el elemento.      |
| `ref`     | string | Referencia exacta del elemento de destino desde la instantánea de la página.                               |

### Tool: `browser_close`
Cierra la página.

### Tool: `browser_console_messages`
Devuelve todos los mensajes de la consola.

### Tool: `browser_drag`
Realiza una operación de arrastrar y soltar entre dos elementos.

**Parámetros:**
| Nombre         | Tipo   | Descripción                                                                                             |
|----------------|--------|---------------------------------------------------------------------------------------------------------|
| `endElement`   | string | Descripción legible por humanos del elemento de destino para obtener permiso para interactuar con el elemento. |
| `endRef`       | string | Referencia exacta del elemento de destino desde la instantánea de la página.                               |
| `startElement` | string | Descripción legible por humanos del elemento de origen para obtener permiso para interactuar con el elemento.  |
| `startRef`     | string | Referencia exacta del elemento de origen desde la instantánea de la página.                                |

### Tool: `browser_file_upload`
Sube uno o varios archivos.

**Parámetros:**
| Nombre  | Tipo  | Descripción                                                                          |
|---------|-------|--------------------------------------------------------------------------------------|
| `paths` | array | Las rutas absolutas a los archivos a subir. Puede ser un solo archivo o varios archivos. |

### Tool: `browser_generate_playwright_test`
Genera una prueba de Playwright para un escenario dado.

**Parámetros:**
| Nombre        | Tipo   | Descripción                          |
|---------------|--------|--------------------------------------|
| `description` | string | La descripción de la prueba.         |
| `name`        | string | El nombre de la prueba.              |
| `steps`       | array  | Los pasos de la prueba.              |

### Tool: `browser_handle_dialog`
Maneja un diálogo.

**Parámetros:**
| Nombre       | Tipo    | Descripción                                                       |
|--------------|---------|-------------------------------------------------------------------|
| `accept`     | boolean | Si se acepta el diálogo.                                          |
| `promptText` | string  | (opcional) El texto del prompt en caso de un diálogo de prompt. |

### Tool: `browser_hover`
Pasa el cursor sobre un elemento en la página.

**Parámetros:**
| Nombre    | Tipo   | Descripción                                                                                             |
|-----------|--------|---------------------------------------------------------------------------------------------------------|
| `element` | string | Descripción legible por humanos del elemento para obtener permiso para interactuar con el elemento.      |
| `ref`     | string | Referencia exacta del elemento de destino desde la instantánea de la página.                               |

### Tool: `browser_install`
Instala el navegador especificado en la configuración. Llama a esto si obtienes un error sobre que el navegador no está instalado.

### Tool: `browser_navigate`
Navega a una URL.

**Parámetros:**
| Nombre | Tipo   | Descripción             |
|--------|--------|-------------------------|
| `url`  | string | La URL a la que navegar. |

### Tool: `browser_navigate_back`
Vuelve a la página anterior.

### Tool: `browser_navigate_forward`
Avanza a la siguiente página.

### Tool: `browser_network_requests`
Devuelve todas las solicitudes de red desde que se cargó la página.

### Tool: `browser_pdf_save`
Guarda la página como PDF.

**Parámetros:**
| Nombre     | Tipo   | Descripción                                                                         |
|------------|--------|-------------------------------------------------------------------------------------|
| `filename` | string | (opcional) Nombre del archivo para guardar el PDF. Por defecto es `page-{timestamp}.pdf`. |

### Tool: `browser_press_key`
Presiona una tecla en el teclado.

**Parámetros:**
| Nombre | Tipo   | Descripción                                                                    |
|--------|--------|--------------------------------------------------------------------------------|
| `key`  | string | Nombre de la tecla a presionar o un carácter a generar, como `ArrowLeft` o `a`. |

### Tool: `browser_resize`
Redimensiona la ventana del navegador.

**Parámetros:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JhonHander) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
