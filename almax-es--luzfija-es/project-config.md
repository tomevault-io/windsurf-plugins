---
trigger: always_on
description: Contexto operativo para agentes que entren al repo de LuzFija.es.
---

# AGENTS.md

Contexto operativo para agentes que entren al repo de LuzFija.es.

## Que Es Este Proyecto

No es solo "un comparador de tarifas". Es una suite frontend local-first con cuatro bloques principales:

1. `/`: comparador principal de tarifas del mercado libre con PVPC, autoconsumo, bateria virtual, bono social, importacion CSV/XLSX y extraccion de factura PDF.
2. `/estadisticas/`: observatorio PVPC y excedentes con historico, KPIs, graficos y analitica personal desde CSV/XLSX.
3. `/comparador-tarifas-solares.html`: simulador independiente mes a mes para tarifas con excedentes remunerados, con o sin BV.
4. Capa editorial y soporte: `guias/`, `guias.html`, `como-funciona-luzfija.html`, `404.html`, `privacidad.html`, `aviso-legal.html`.

Tambien es importante lo que no es: no monetiza el ranking, no vende leads y no tiene referidos, comisiones, publicidad ni acuerdos comerciales que condicionen el orden de resultados.

## Orden Recomendado De Lectura

1. `CAPACIDADES-WEB.md`
2. `AUDITORIA-IA.md` si vas a hacer revision, auditoria o threat modeling. Lee completo `AUDITORIA-IA.md`; consulta `AUDITORIA-REGISTRO.md` unicamente por el area relevante, no de forma lineal.
3. `ARRANQUE-CARGA.md` **obligatorio antes de tocar el orden de carga**: etiquetas `<script>`, `defer`/`async`, hojas de estilo, preloads o el registro del service worker
4. `README.md`
5. `ARQUITECTURA-CALCULOS.md`
6. `CALC-FAQS.md`
7. `MANTENIMIENTO-NORMATIVO.md`
8. `SIMULADOR-BV.md`
9. `ANALITICA-GOATCOUNTER.md`
10. `JSON-SCHEMA.md`
11. `PVPC-SCHEMA.md`
12. `llms.txt` y `llms-full.txt` para ver como se presenta la herramienta a asistentes externos

`CAPACIDADES-WEB.md` es la fuente de verdad funcional. Si algo parece contradecir otra doc, parte de ahi.

`ARRANQUE-CARGA.md` es la fuente de verdad del contrato de arranque. Varios modulos desestructuran sus dependencias en tiempo de evaluacion, asi que el orden de los recursos en el HTML es contrato de ejecucion, no formato. Algunas regresiones de arranque no producen excepcion ni senal diagnostica automatica, aunque sus consecuencias funcionales si sean observables; las vigila `tests/bootstrap-contract.test.js`.

## Mapa Rapido Del Codigo

- `index.html` + `js/lf-*.js`: comparador principal, estado, inputs, calculo, render, CSV y cache.
- `js/pvpc.js`: motor PVPC usando datasets locales en `data/pvpc/`.
- `js/factura.js`: extraccion PDF, carga de jsQR y OCR opcional con Tesseract; los parsers de texto y QR CNMC viven en `js/factura-parsers.js`.
- `js/desglose-*.js`: desglose detallado de factura en la home.
- `estadisticas/index.html` + `js/pvpc-stats-*.js`: observatorio PVPC/excedentes.
- `comparador-tarifas-solares.html` + `js/bv/*.js`: simulador solar/BV y flujo hibrido CSV -> manual.
- `js/lf-csv-utils.js`: parser horario compartido y clasificacion P1/P2/P3 canonica.
- `js/tracking.js` + `vendor/goatcounter/count.js`: analitica GoatCounter, pageviews canonicos, eventos, privacidad y saneo de referrers. Ver `ANALITICA-GOATCOUNTER.md`.
- `sw.js`: cache/PWA/update flow.
- `scripts/sync-seo-docs.mjs`: sincroniza sitemap e indice de busqueda y, con `--include-repo-docs`, tambien metricas/referencias de README, CAPACIDADES, JSON-SCHEMA y `llms*`, mas el indice de `AUDITORIA-IA.md` generado desde `AUDITORIA-REGISTRO.md`.
- `scripts/check_data_freshness.py`: guardia de frescura de los datasets (pvpc/surplus/ssaa); `pvpc.yml` la ejecuta tras la descarga diaria e incluye self-test (`--self-test`). Detalle en `PVPC-SCHEMA.md`.

### Inventario Completo De Modulos JS

Una linea por modulo para no confundir ficheros con nombres parecidos (`config.js` vs `lf-config.js`) ni asumir que un modulo pequeno es prescindible.

| Modulo | Proposito |
| --- | --- |
| `js/config.js` | Guard global defensivo (define `currentYear` y globals legacy antes que el resto de scripts). No confundir con `lf-config.js`. |
| `js/error-bootstrap.js` | Buffer efimero y sin mensajes para errores first-party tempranos + watchdog visual de ultimo recurso cuando no llega a cargar el coordinador completo de home, factura, desglose, solar u observatorio. Deja ademas una solicitud cerrada de recuperacion para que el coordinador del SW ofrezca recarga aunque falle tracking. Se carga antes de `config.js` en las tres aplicaciones. |
| `js/theme.js` | Guard redundante + tema temprano para entradas legacy que cargan `theme.js` antes que nada. |
| `js/shell-lite.js` | Tema + menu para paginas sin `lf-app`/`bv-ui` (guias, landings, legal, 404). |
| `js/lf-sw-update.js` | Registro + auto-update + guard de recarga del service worker (`window.LF.initSwUpdate`), compartido por `lf-app.js` y `shell-lite.js`; ante una dependencia esencial ausente fuerza `update()`, compara builds y ofrece recarga explicita. Debe cargarse antes que ellos en el HTML. |
| `js/index-extra.js` | Scripts de la home extraidos de `index.html` (modal PVPC, instalacion PWA, compartir) para cacheo y CSP. |
| `js/index-extra-loader.js` | Shim de compatibilidad para clientes con SW/HTML antiguos que aun piden ese fichero; retirable cuando dejen de solicitarlo. |
| `js/inp-debug.js` | Instrumentacion INP, solo activa con `?debug=1` o `lf_debug=1`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [almax-es/luzfija.es](https://github.com/almax-es/luzfija.es) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
