---
trigger: always_on
description: Un catálogo de fuentes de datos de la Administración pública española, escrito para que lo consuman agentes de IA
---

# Instrucciones para agentes que trabajan en este repo

## Qué es esto

Un catálogo de fuentes de datos de la Administración pública española, escrito para que lo consuman agentes de IA
y desarrolladores que construyen encima. No es documentación divulgativa. Es un mapa operativo: qué hay, dónde
está, cómo se llama, qué devuelve, qué falla.

## Objetivo que manda sobre todo lo demás

Máxima utilidad para construir, investigar y desarrollar sobre datos públicos, con el mínimo de tokens.
Cada línea que no ahorre una búsqueda, una prueba fallida o una hora de depuración a quien la lea, sobra.

## Reglas de contenido

1. **Solo lo esencial.** Una fuente entra si un builder la usaría. Un dato entra en la ficha si cambia cómo se
   programa contra la fuente. Historia del organismo, adjetivos, contexto institucional: fuera.
2. **Verificar antes de escribir.** Toda URL, endpoint, parámetro y formato se prueba con una llamada real
   antes de afirmarse. Si responde, `verified` lleva la fecha de hoy. Si no se puede probar, `verified: null`
   y se dice por qué en `gotchas`. Nunca inventar endpoints ni parámetros plausibles. Un intento fallido de
   automatizar no demuestra que no se pueda: se escribe «no localizado» o «no conseguido», con lo probado y la
   fecha, nunca «no existe» o «no es posible», salvo que lo diga la documentación oficial o el propio servidor
   (404, 410, 401).
3. **Las trampas son el valor.** `gotchas` recoge lo que la documentación oficial no dice: cabeceras
   obligatorias, codificaciones, decimales con coma, límites no documentados, ids que no coinciden entre
   organismos, URLs que cambian, datos que parecen cero y son secreto estadístico. Una frase por trampa.
4. **`tips` solo si acelera.** Patrón de uso, librería concreta, cruce típico con otra fuente. Máximo seis.
5. **Ejemplos copiables y respuesta descrita.** Cada endpoint principal lleva un `example` que funciona al
   pegarlo y un `returns` con la forma de la respuesta vista en esa llamada (campos clave, tipos, formato de fecha y
   decimal, paginación), nunca copiada de la documentación. Con claves, usar variable de entorno (`$AEMET_KEY`),
   nunca una clave real.
6. **Vocabulario cerrado.** Sector, acceso, auth, periodicidad, formatos, estado, quirks e ids salen de `schema/vocab.yaml`.
   Si falta un valor, se añade al vocabulario en el mismo commit, no se improvisa.
7. **Castellano en valores, inglés en claves.** Sin markdown dentro de los valores. Sin dos puntos seguidos de
   espacio en valores sin comillas, porque rompe el YAML.
8. **Fuente única de verdad.** Solo se editan `sources/**/*.yaml`, `indices/*.yaml`, `guides/*.md`, `schema/`,
   `scripts/` y `evals/`, más los ficheros de distribución del servidor MCP (`pyproject.toml`, `server.json`, `glama.json`,
   `Dockerfile`, `mcpb/`) y `.github/`. `catalog.json`, `llms.txt`, `llms-full.txt`, `indices/README.md`, los `README.md` de sector y la
   tabla del README raíz se regeneran con `python scripts/build.py` y se suben en el mismo commit.
9. **No borrar fichas.** Una fuente muerta pasa a `status: deprecated` con la sustituta en `gotchas`.
10. **Rendimientos decrecientes.** Si un sector solo tiene portales sin API y datos que ya da el INE, una
    ficha o ninguna. Mejor 50 fichas exactas que 500 aproximadas.

## Flujo de trabajo

```bash
pip install -r scripts/requirements.txt
python scripts/validate.py        # esquema, vocabulario, ids, referencias
python scripts/build.py           # regenera todo lo derivado
python scripts/check_links.py     # informe de URLs (necesita red)
python scripts/check_recetas.py   # batería de regresión de las recetas (necesita red; --report, --fail)
python scripts/check_ejemplos.py  # ejecuta el example de cada endpoint de las fichas (necesita red; --only, --muestra, --report, --fail)
python scripts/test_clientes.py   # parsers de scripts/clientes contra las muestras reales, sin red (corre en CI)
python scripts/fnmt_bundle.py     # genera ca-age.pem (certifi + CA de FNMT) para los hosts con cadena incompleta
python scripts/mcp_catalogo.py    # servidor MCP por stdio sobre catalog.json (guides/servidor-mcp.md); prueba real con test_mcp_catalogo.py, fuera de CI
```

Antes de cada commit: validate y build limpios. Commits pequeños por sector o por lote verificado.
Sin subagentes salvo petición expresa: el trabajo es secuencial y de precisión.

## Índices agregados (`indices/`)

Cinco ficheros que responden a lo que una ficha sola no responde; `validate.py` comprueba que solo citan ids
de fichas y del vocabulario, y `build.py` los vuelca en `indices/README.md`, `llms.txt` y `catalog.json`.

- `recetas.yaml`: procedimiento por intención que encadena fichas. Entra una receta si cruza dos o más fuentes
  o si la vía directa esconde una trampa. Cada paso cita una ficha; cada receta lleva al menos un `check`
  (URL, cabeceras, texto esperado) que `check_recetas.py` ejecuta como regresión. `verified` con fecha solo si
  todos los pasos se probaron; si no, `null` y `note` con lo que falta.
- `necesidades.yaml`: una línea por necesidad habitual con la ficha que la resuelve y la nota que evita el

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BquantFinance/Administracion-fuentes-publicas](https://github.com/BquantFinance/Administracion-fuentes-publicas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
