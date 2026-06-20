---
trigger: always_on
description: >
---


# BOE CLI — Guía de M&A para Agentes IA

## ¿Qué es el BOE CLI?

El BOE CLI (`./boe`) es una herramienta Go que consulta la API de datos abiertos del BOE español para obtener legislación consolidada vigente. Devuelve el texto actualizado de leyes, incluyendo todas sus modificaciones históricas anotadas.

**Ubicación del binario:** `/Users/santiagoquintana/clawd/projects/boe-cli/boe`

---

## Comandos Disponibles

```bash
# Buscar legislación
./boe search "query"                          # búsqueda general
./boe search "texto:palabra"                  # busca en el texto
./boe search "titulo:nombre ley"              # busca en el título

# Consultar una ley específica
./boe law BOE-A-XXXX-XXXXX                    # metadatos de la ley
./boe law BOE-A-XXXX-XXXXX --index            # índice/estructura de la ley
./boe law BOE-A-XXXX-XXXXX --block <id>       # artículo/bloque específico

# Ejemplos reales
./boe law BOE-A-2010-10544 --block a107       # LSC art. 107
./boe law BOE-A-2015-11430 --block a44        # ET art. 44
./boe law BOE-A-2014-12328 --block a21        # LIS art. 21
```

### Notas sobre búsquedas:
- Búsquedas multi-palabra se expanden automáticamente con AND
- Usa `titulo:` para afinar a leyes específicas (menos ruido)
- Los IDs de bloques suelen seguir el patrón `a<número>` (art. 107 → `a107`)
- Algunos artículos tienen IDs especiales: `a348bis`, `a15-2` (art. 15 bis)
- El CLI devuelve JSON; el texto del artículo está en el campo `text`

---

## Marco Legal M&A en España: Referencia Completa

### Leyes Core de M&A

| Ley | BOE ID | Descripción |
|-----|--------|-------------|
| **LSC** — Ley de Sociedades de Capital | `BOE-A-2010-10544` | Derecho corporativo; SL y SA; transmisión de participaciones, quórums, separación |
| **ET** — Estatuto de los Trabajadores | `BOE-A-2015-11430` | Sucesión de empresa, contratos laborales, convenios colectivos |
| **LME** — Modificaciones Estructurales | `BOE-A-2023-15135` | Fusiones, escisiones, transformaciones, cesiones globales (RDL 5/2023) |
| **LDC** — Ley de Defensa de la Competencia | `BOE-A-2007-12946` | Control de concentraciones, umbrales CNMC, notificación obligatoria |
| **CC** — Código Civil | `BOE-A-1889-4763` | Contratos, obligaciones, base del SPA, resolución, saneamiento |
| **CdeC** — Código de Comercio | `BOE-A-1885-6627` | Grupos de sociedades (art. 42), contabilidad, deberes mercantiles |

### Leyes Fiscales

| Ley | BOE ID | Descripción |
|-----|--------|-------------|
| **LIS** — Impuesto sobre Sociedades | `BOE-A-2014-12328` | Régimen fiscal neutral fusiones (art. 76-89), exención participaciones (art. 21) |
| **LIRPF** — IRPF | `BOE-A-2006-20764` | Ganancias patrimoniales para vendedores personas físicas (art. 33) |
| **LITPAJD** — ITP y AJD | `BOE-A-1993-25359` | Exención transmisión acciones, excepción inmobiliaria anti-fraude |
| **LGT** — Ley General Tributaria | `BOE-A-2003-23186` | Prescripción, responsabilidad tributaria, recargos |

### Leyes Regulatorias y Otras

| Ley | BOE ID | Descripción |
|-----|--------|-------------|
| **TRLC** — Ley Concursal | `BOE-A-2020-4859` | Insolvencia, acciones rescisorias, acuerdos de refinanciación |
| **LMV** — Ley Mercado de Valores | `BOE-A-2023-22765` | OPAs, exención ITP transmisión valores, información privilegiada |
| **LIVA** — IVA | `BOE-A-1992-28740` | Exención art. 20.18 en transmisión de valores; IVA en asset deals |
| **Blanqueo** — Ley 10/2010 | `BOE-A-2010-6737` | Obligaciones KYC para asesores, identificación del beneficiario real |
| **LOPDGDD** — Protección de Datos | `BOE-A-2018-16673` | Due diligence RGPD, transferencia de datos en M&A, DPO |
| **Inversiones Exteriores** | `BOE-A-1999-2725` | Screening de inversiones extranjeras (mecanismo de control desde 2020) |

---

## Artículos Clave por Área M&A

### 1. TRANSMISIÓN DE PARTICIPACIONES / SHARE DEAL

#### LSC — Régimen de transmisión

| Artículo | Bloque CLI | Qué cubre | Por qué importa en M&A |
|----------|-----------|-----------|------------------------|
| Art. 107 | `a107` | Régimen de transmisión voluntaria de participaciones SL inter vivos | **Crítico**: sistema de consentimiento de JG + derecho de adquisición preferente de socios; cualquier SPA de SL lo debe respetar o estatutos deben modificarlo |
| Art. 108 | `a108` | Cláusulas estatutarias prohibidas | Nulas cláusulas que hagan "prácticamente libre" la transmisión; lock-up máx. 5 años |
| Art. 123 | `a123` | Transmisión forzosa de participaciones | Régimen para ventas judiciales/embargos |
| Art. 124 | `a124` | Transmisión mortis causa | Sucesión; estatutos pueden establecer régimen especial |

**Flujo de una transmisión SL sin previsión estatutaria especial:**
1. Socio comunica a administradores: nº participaciones, identidad adquirente, precio y condiciones
2. JG (mayoría ordinaria) vota: consiente o deniega
3. Si deniega: debe identificar adquirente alternativo (socios preferentes)
4. Si no comunica adquirente en 3 meses → libre transmisión
5. Precio en operaciones a título oneroso distinto de compraventa: experto independiente

#### LSC — Competencias JG y quórums

| Artículo | Bloque CLI | Qué cubre | Por qué importa en M&A |
|----------|-----------|-----------|------------------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zepelinmad/boe-cli](https://github.com/zepelinmad/boe-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
