---
trigger: always_on
description: Material del curso **Elaboración de libros electrónicos mediante código y asistentes de Inteligencia Artificial** y plantilla para que docentes NO informáticos creen libros interactivos con Jupyter Book / TeachBooks.
---

# Elaboración de libros electrónicos mediante código y asistentes de Inteligencia Artificial — Instrucciones para el Agente

## Objetivo del Proyecto

Material del curso **Elaboración de libros electrónicos mediante código y asistentes de Inteligencia Artificial** y plantilla para que docentes NO informáticos creen libros interactivos con Jupyter Book / TeachBooks.
El usuario típico es un profesor de las Facultades de Ciencias y de Ciencias Químicas (USAL) que **nunca ha usado un terminal**.
Tu trabajo es que TODO funcione con fricción CERO.

## Reglas de Oro

1. **SIEMPRE** usa el entorno virtual `.venv`. Nunca instales paquetes en el Python global.
2. **Requiere Python 3.12**. `scripts/setup_env.py` crea `.venv` con Python 3.12 mediante `uv`; si `.venv` existe con otra versión, para y muestra diagnóstico.
3. **Acción > Explicación**. Ejecuta los scripts, no des clases de informática.
4. **Responde en español** siempre (salvo que te pidan lo contrario).
5. **No añadas frameworks JS** (React, Vue, etc.). El sistema de build es Jupyter Book, punto.
6. **No rompas la estructura multi-idioma**. Si añades contenido en un idioma, DEBE existir en TODOS los idiomas configurados.
7. **No crees entornos alternativos** (`.venv_linux`, `.venv2`, `/tmp/venv`, etc.). Si `.venv` no corresponde al sistema actual, para y muestra diagnóstico.
8. **Codificación UTF-8 siempre**. Todos los `.md`, `.ipynb`, `.yml`, `.py`, `.tex`, `.bib`, `.js`, `.css` y skills deben guardarse como UTF-8. Nunca escribas texto con mojibake, caracteres de reemplazo (`U+FFFD`) ni sustituyas acentos por `?`. Antes de cerrar cambios de contenido, ejecuta `python scripts/check_encoding.py` o `python scripts/check_multilang_integrity.py`.
9. **Celdas de código ASCII-safe**. En notebooks, el Markdown puede llevar acentos, pero las celdas `code` deben usar ASCII en comentarios, `print()`, títulos/ejes/leyendas de Matplotlib y strings visibles. Usa `Ohm`, `pi`, `Delta`, `uF`, `tau`, `x^2` o mathtext (`r"$x^2$"`) en lugar de símbolos Unicode dentro del código.
10. **Assets web seguros**. PNG/JPG/SVG son formatos estables para HTML/PDF. No cambies referencias a WebP de forma general salvo que exista fallback PNG/JPG y se haya probado el PDF. Tras añadir imágenes o GIFs, ejecuta `python scripts/optimize_static_assets.py --check`; usa `--fix` solo para optimización conservadora de PNG/JPG.

## Arquitectura del Proyecto

```
teachbook_usal_template/
├── book/                          # Contenido del libro
│   ├── _config_es.yml             # Configuración español (idioma principal)
│   ├── _config_en.yml             # Configuración inglés
│   ├── _toc_es.yml                # Tabla de contenidos español
│   ├── _toc_en.yml                # Tabla de contenidos inglés
│   ├── _static/                   # Assets estáticos (CSS, JS, logos, PDFs)
│   │   ├── custom.css
│   │   ├── custom.js
│   │   ├── logo.png
│   │   ├── usal_logo.png
│   │   └── references.bib         # Bibliografía BibTeX
│   ├── es/                        # Contenido en español
│   │   ├── intro.md
│   │   ├── 01_tutorial/           # Sección tutorial
│   │   ├── 02_grados/             # Sección ejemplos por grado
│   │   ├── 90_acerca_de.md
│   │   ├── 91_licencias.md
│   │   └── 92_como_citar.md
│   └── en/                        # Contenido en inglés (misma estructura)
├── scripts/                       # Scripts de automatización
│   ├── setup_env.py               # Configuración del entorno
│   ├── build_book.py              # Compilación HTML
│   ├── launch_preview.py          # Lanzador seguro de vista previa para personas/agentes
│   ├── preview_book.py            # Vista previa usando el MISMO build que producción
│   ├── export_pdf.py              # Exportación a PDF
│   ├── setup_latex.py             # Instalación de LaTeX (Tectonic)
│   └── git_helper.py              # Guardar y publicar
├── latex_templates/               # Plantillas LaTeX personalizadas
│   ├── common/                    # Estilos compartidos (jupyterBook.cls)
│   ├── es/                        # Ajustes español (language_support.tex)
│   └── en/                        # Ajustes inglés
├── .github/
│   ├── skills/                    # Skills del proyecto (FUENTE DE VERDAD)
│   └── workflows/deploy.yml       # GitHub Actions: build + deploy
├── AGENTS.md                      # Este archivo
└── requirements.txt               # Dependencias Python
```

## Dependencias por Capas

La instalación inicial debe ser ligera. No mezcles dependencias científicas o de importación PDF en la base.

| Archivo | Cuándo se instala | Contenido |
|---|---|---|
| `requirements.txt` | Siempre | Build HTML, preview, citas, Kroki, Thebe y assets básicos |
| `requirements-pdf.txt` | `--extras pdf` | Exportación PDF y conversión SVG para LaTeX |
| `requirements-notebooks.txt` | `--extras notebooks` | Ejecución local de notebooks científicos |
| `requirements-pdf-import.txt` | `--extras pdf-import` | Conversión de PDFs a Markdown; puede arrastrar paquetes pesados |
| `requirements-dev.txt` | `--dev` | Tests y herramientas de desarrollo |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juanjo810/libro_computadores_i](https://github.com/juanjo810/libro_computadores_i) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
