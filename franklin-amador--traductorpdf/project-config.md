---
trigger: always_on
description: Contexto completo del proyecto para continuar trabajo sin leer el historial completo.
---

# AGENTS.md — Guía para agentes de IA

Contexto completo del proyecto para continuar trabajo sin leer el historial completo.

---

## Qué hace este proyecto

Traductor de PDF 100% local y gratuito. El usuario elige un PDF, selecciona los idiomas
y obtiene un PDF nuevo con el texto traducido pero el layout, imágenes y colores intactos.
Sin APIs externas, sin internet (después de la descarga inicial del modelo), sin cuentas.

---

## Stack técnico

| Módulo | Librería | Por qué esta elección |
|--------|----------|-----------------------|
| Extracción PDF | `pymupdf` (fitz) | Accede a bounding boxes, spans y flags de fuente |
| Traducción | `ctranslate2` | 3-5× más rápido que transformers en CPU, int8 |
| Tokenización | `sentencepiece` | Tokenizador nativo de los modelos MarianMT |
| Descarga modelos | `huggingface_hub` | Solo en primera ejecución |
| GUI | `tkinter` / `ttk` | Incluido con Python, sin dependencias extra |

**Instalación:**
```
pip install pymupdf ctranslate2 sentencepiece huggingface_hub
```

---

## Archivos y responsabilidades

```
Traductor/
├── main.py       # Único: instancia Tk y TranslatorApp, lanza mainloop
├── gui.py        # TranslatorApp: widgets, threading, queue de mensajes UI↔worker
├── extractor.py  # Abre PDF, extrae bloques con bbox/spans, de-hyphenation
├── translator.py # Carga modelos CT2, split de oraciones, batch translate, anti-loops
├── glossary.py   # Glosario EN→ES de títulos exactos (bypassea el modelo)
├── builder.py    # Redacta texto original, reinserta traducción con tipografía original
├── eval.py       # Harness A/B: run/diff/scan sobre el PDF de referencia
├── _test_dehyphen.py # Tests de de-hyphenation (requiere el PDF de referencia)
└── _test_compact.py  # Tests de _compact_heading y _has_dup_prefixes
```

**Regla de dependencias:** `gui.py` importa los otros tres. Los módulos `extractor`,
`translator` y `builder` son independientes entre sí y no importan `gui`.

---

## Flujo completo (trazado de datos)

```
PDF en disco
    │
    ▼ extractor.extract_blocks(pdf_path)
list[list[dict]]   ← páginas → bloques {bbox, text, spans, page_index}
    │
    │ aplanar: flat_texts = [b["text"] for page in all_pages for b in page]
    ▼
translator.translate_batch(flat_texts, src_sp, tgt_sp, ct2, progress_cb)
    │
    │  internamente:
    │  1. filtrar triviales (_needs_translation)
    │  2. _split_sentences  → oraciones individuales
    │  3. _normalize        → ASCII equiv. de Unicode fancy
    │  4. src_sp.encode     → tokens SentencePiece
    │  5. sort by len       → batches homogéneos
    │  6. ct2.translate_batch (beam=2, ngram=3, rep_pen=1.5, max_tgt escalonado)
    │  7. tgt_sp.decode     → texto
    │  8. _truncate_output  → cortar continuación espuria
    │  9. rejoin sentences  → " ".join por bloque original
    ▼
list[str]  (misma longitud que flat_texts)
    │
    │ reagrupar por página
    ▼
builder.build_translated_pdf(pdf_path, all_pages, translated_pages, output_path)
    │
    │  por página:
    │  1. add_redact_annot + apply_redactions(PDF_REDACT_IMAGE_NONE)
    │  2. _select_font   → fuente PDF estándar según flags/nombre
    │  3. _detect_align  → LEFT / CENTER / JUSTIFY
    │  4. _insert_fitting → retry geométrico (−20%/intento, máx 8)
    ▼
PDF guardado en disco
```

---

## translator.py — Decisiones de diseño críticas

### Por qué sentence-level y no block-level

Los modelos OPUS-MT fueron entrenados con pares de oraciones (corpus paralelos como
OPUS). Un bloque de párrafo de 80-150 tokens supera el rango de entrenamiento y provoca
que el decoder genere loops ("del del del", "diseño diseño diseño").

`_split_sentences` usa `_SENT_RE = re.compile(r'(?<=[.!?])\s+(?=[A-Z"\(\[])')` para
dividir antes de traducir y `" ".join(translated_sents)` para reunir después.

### Cap `max_tgt` escalonado — por qué importa

CTranslate2 no para en EOS automáticamente para entradas cortas; llena el budget.
Con el cap constante antiguo (`max_src + 25`), "2nd Edition" (3 tokens) recibía
28 tokens de margen — suficiente para 9 iteraciones de "Edición 2a edición segunda...".

| max_src del batch | Fórmula | Caso típico |
|---|---|---|
| ≤ 5 tokens | `max_src + 4` | Nombres, headings de 1-2 palabras |
| 6–20 tokens | `max_src + 8` | Oración típica |
| > 20 tokens | `min(max_src + 15, 200)` | Oración larga / compleja |

**Nota sobre batching:** las secuencias se ordenan por longitud antes de agrupar
(bucket batching). En un PDF con miles de frases, las frases cortas forman su propio
batch con `max_src` pequeño. En tests de < 32 frases todas caen en un batch y el
comportamiento se degrada ligeramente — es esperado y no un bug.

### `_truncate_output` — por qué existe

El modelo a veces no emite EOS y genera una segunda traducción pegada a la primera.
Como cada INPUT es una sola oración, el primer boundary de oración en el OUTPUT es
siempre espurio. Tres patrones en orden de prioridad:

```python
_CONT_RE       = r'(?<=[.!?])[ \t]*(?=[A-Z])'       # ". Capital" o ".Capital"
_CONT_LOWER_RE = r'(?<=[.!?])[ \t]+(?=[a-záéíóúüñ]{2,})'  # ". minúscula"
_DOT_DOT_RE    = r'(?<=[.!?])[ \t]*\.'               # ". ." — ruido de tokens
```

Se toma el match más temprano y se devuelve `text[:match.start()]` (la posición

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Franklin-Amador/TraductorPDF](https://github.com/Franklin-Amador/TraductorPDF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
