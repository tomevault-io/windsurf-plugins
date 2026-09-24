---
trigger: always_on
description: App de escritorio PySide6 (Qt 6) multiplataforma para la elaboración de **presupuestos de obra** (ingeniería y arquitectura): análisis de costos unitarios (ACU), cronograma Gantt valorizado con ruta crítica (CPM), metrados (incluido acero), fórmula polinómica e índices INEI, Control de Obra y 13 reportes profesionales.
---

# IngePresupuestos

App de escritorio PySide6 (Qt 6) multiplataforma para la elaboración de **presupuestos de obra** (ingeniería y arquitectura): análisis de costos unitarios (ACU), cronograma Gantt valorizado con ruta crítica (CPM), metrados (incluido acero), fórmula polinómica e índices INEI, Control de Obra y 13 reportes profesionales.

**Autor:** Ing. Marco Sumari · **Software libre GPL-3.0-or-later** · Próxima versión: **3.0** (retorno a software libre)

> **IngePresupuestos es software libre.** Todo el código está bajo **GPL-3.0-or-later** (`LICENSE`). No hay funciones de pago, ni trial, ni registro: la app completa es gratuita para todos. El apoyo es **voluntario** — Yape/Plin en Perú y [Liberapay](https://liberapay.com/ingelibre/donate) desde el extranjero.
>
> **El cierre de agosto de 2026 se revirtió.** La 2.9.0 se publicó como propietaria (release y `version.json` del 8 de agosto); el 14 de agosto se decidió volver a lo libre y la 3.0 sale bajo GPL. Motivo: con ~20 usuarios ningún modelo de cobro producía ingreso, mientras que el modelo cerrado sí costaba tiempo (difusión, soporte, emisión manual de claves) — y además cerraba el acceso a firma de código gratuita para proyectos OSS.
>
> **No queda maquinaria de licencias.** Se eliminaron `core/licencia.py`, `views/licencia_dialog.py`, `scripts/gen_license.py`, `generar-licencia.sh` y `resources/license_public.pem`, junto con los 22 gates `require_premium`/`_gate_reporte_editable` y el candado de descargas de `update_manager`. **No reintroducir candados.**
>
> Las versiones ≤2.8.8 siguen archivadas en `backups/gpl-archivo-2.8.x/` (ver su `LEEME.md` — la obligación GPLv3 §6 sigue vigente hasta 2029; conservar aunque el repo ya sea público).
>
> **Copyright a nombre de Marco Sumari** (persona, no Sumari SAC): es lo que conserva la libertad de relicenciar. **Antes de fusionar un PR externo hace falta CLA** o el copyright deja de ser único.
>
> Al tocar dependencias o recursos, actualizar `THIRD-PARTY-NOTICES.txt`. **El build DEBE seguir siendo `onedir`** — es requisito de la LGPL-3.0 de Qt (ver notas). El changelog detallado vive en `git log`.

Web: `ingepresupuestos.com` · Docs: `docs.ingepresupuestos.com`

---

## Entorno

```bash
# Python 3.12+ · PySide6 6.x
cd /home/sumaritux/Proyectos/ingepresupuestos/app
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
python3 main.py          # o ./iniciar.sh   (Wayland: INGEPPTO_FORCE_XCB=1 fuerza xcb)
```

Tests sin GUI (usan copia temporal de `presupuestos_seed.db`, nunca la BD activa):
```bash
QT_QPA_PLATFORM=offscreen venv/bin/python3 tests/test_reglas_negocio.py   # reglas de negocio
venv/bin/python3 tests/test_core.py
# también: test_almacen.py · test_curva_s.py · test_valorizacion.py · test_catalogos.py · test_navegacion.py · test_pdf_escala_texto.py · test_formato_reporte.py
```

---

## Arquitectura

| Capa | Tecnología | Carpeta |
|------|-----------|---------|
| UI | PySide6 6.x (Qt 6) + QtPdf/QtPdfWidgets | `views/`, `widgets/` |
| Backend | Python 3 puro | `core/`, `utils/` |
| BD | SQLite 3 (`presupuestos.db`) | — |
| Reportes PDF | QTextDocument + QPdfWriter + QPainter | `core/pdf_reports.py` |
| Reportes Word | python-docx | `core/word_reports.py` |
| Reportes ODT/ODS | LibreOffice headless (conversión) | `core/odt_reports.py`, `core/ods_reports.py`, `core/soffice.py` |
| Excel | openpyxl | `core/exporter.py` |
| Importación | openpyxl + xlrd + pdfplumber + mdbtools/pyodbc | `core/importer.py` y siblings |
| IA (opcional) | Anthropic/Groq/OpenRouter/Gemini/OpenAI/Ollama | `core/ai_specs.py` |
| Fuzzy / RAG | rapidfuzz + model2vec int8 (sin PyTorch) | `core/asistente_local.py`, `core/biblioteca_embeddings.py` |
| Empaquetado | PyInstaller 6 + GitHub Actions | `ingepresupuestos.spec`, `.github/workflows/` |

Rutas (`core/config.py`): `BASE_DIR` (read-only; bajo PyInstaller = `_internal/`), `USER_DATA_DIR` (Linux `~/.local/share/ingepresupuestos/`, Windows `%APPDATA%/ingepresupuestos/`, macOS `~/Library/Application Support/…`), `DB_PATH = USER_DATA_DIR/presupuestos.db`. `_sembrar_db_si_falta` copia el seed solo si la BD no existe.

`main.py` NO procesa `sys.argv` para abrir un archivo pasado (la asociación de `.db` es solo cosmética — ver abajo).

---

## Reglas críticas de negocio (NO romper)

```python
# Precios por proyecto — siempre COALESCE
COALESCE(ai.precio, r.precio, 0)

# Cantidad MO en ACU — y equipo por hora (unidad hh/hm/he): se DERIVA de la cuadrilla.
#   Helpers en core/database.py: recurso_por_hora · recurso_por_dia · partida_global.
#   UNA sola definición (2026-08-29): antes vivían tres veces, con un comentario
#   que pedía «mantener en sync» a mano. Las vistas las importan con su nombre
#   local (_recurso_por_hora en proyecto_view, _es_por_hora en el selector).
#   NO volver a copiarlas: deciden la cantidad de MO de TODO el presupuesto.
cantidad = (cuadrilla / rendimiento) * jornada_laboral
# `he` (hora-equipo) entró el 6 sep 2026: son 12 equipos del seed que quedaban con

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ingelibre/ingepresupuestos](https://github.com/ingelibre/ingepresupuestos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
