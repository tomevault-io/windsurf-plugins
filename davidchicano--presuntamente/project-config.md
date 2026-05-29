---
trigger: always_on
description: Guía para cualquier agente LLM (Claude Code, Codex, Cursor, etc.) que trabaje en este repositorio. Este fichero es la fuente; `CLAUDE.md` es un symlink a él.
---

# AGENTS.md — presuntamente

Guía para cualquier agente LLM (Claude Code, Codex, Cursor, etc.) que trabaje en este repositorio. Este fichero es la fuente; `CLAUDE.md` es un symlink a él.

## Qué es este proyecto

**presuntamente** es un inventario interactivo, público y open source de los casos de corrupción más relevantes en España. La misión es **reducir desinformación** ofreciendo una referencia objetiva, trazable y sin cuota política de los procedimientos judiciales relevantes.

- Sitio: `presuntamente.org` (dominio registrado en Cloudflare Registrar el 2026-05-23; canales `contacto@`, `rectificacion@` y `aportar@` operativos vía Cloudflare Email Routing. Publicación técnica vía Cloudflare Pages; DNS apex y `www` se activan desde el panel de Cloudflare cuando el maintainer lo decida).
- Licencia código: AGPL-3.0.
- Licencia contenido editorial: CC BY-SA 4.0.

## Principios irrenunciables

Cualquier acción que hagas debe respetar:

1. **Imputación ≠ condena.** Modelado como `RolEnCaso` con tipos discretos (`investigado`, `procesado`, `acusado`, `condenado`, `absuelto`, `desimputado`) y trayectoria temporal. Nunca afirmaciones que insinúen culpabilidad sin sentencia firme.
2. **Cada afirmación con su fuente y nivel.** Los `Hecho` exigen `Documento` de respaldo con nivel (1-4) visible.
3. **Tratamiento sin cuota política.** Casos vivos o cerrados, de cualquier partido, ordenados por relevancia objetiva.
4. **Presunción de inocencia en el lenguaje.** Verbos prohibidos para hechos no acreditados ("robó", "estafó", "se apropió"). Preferidos: "se investiga", "se atribuye", "consta en el informe X que…".
5. **No exposición innecesaria de personas privadas.** Sólo si tienen rol formal en el procedimiento. V-17: revisión obligatoria de anonimización cuando cierran sus roles.
6. **El git log es el changelog público.** Nunca borres información; corrige con `corregido_por` y conserva el histórico.

## Documentos de diseño

Toda decisión arquitectónica, editorial o legal está justificada en `docs/diseno/`:

- `01-modelo-de-datos.md` — entidades, atributos, 21 reglas CI (V-01..V-21).
- `02-ficha-de-caso.md` — secciones, citación inline, 10 reglas anti-desinformación (P-01..P-10).
- `03-estrategia-de-mantenimiento.md` — watchers, pipeline, uso de LLM con guardarraíles.
- `04-riesgos-legales-y-eticos.md` — marco legal, disclaimer, protocolo querella.
- `05-arquitectura-tecnica.md` — stack.
- `06-roadmap-por-fases.md` — fases ejecutables.

Si vas a tocar algo no trivial, consulta primero el doc correspondiente.

## Estructura del repositorio

```text
/AGENTS.md                  ← este fichero
/CLAUDE.md                  ← symlink → AGENTS.md
/ROADMAP.md                 ← estado operativo vivo del proyecto (leer al iniciar, actualizar al cerrar)
/DESIGN.md                  ← lenguaje visual canónico (Claude Design + Claude Code)
/README.md                  ← descripción público-facing
/CONTRIBUTING.md            ← cómo contribuir
/LICENSE                    ← AGPL-3.0 (código)
/LICENSE-CONTENT.md         ← CC BY-SA 4.0 (contenido editorial)
/LEGAL.md                   ← aviso legal (placeholder hasta publicación con dominio)
/docs/diseno/               ← documentos de diseño
/docs/roadmap/              ← histórico largo, fases cerradas y aprendizajes no canónicos del roadmap
/docs/web/pages/            ← backlog y notas por página visible del sitio (uno por ruta)
/docs/web/features/         ← ficha por feature transversal (feed, búsqueda, RichProse, archive.org, …)
/content/                   ← contenido canónico (YAML)
  casos/<slug>/
    caso.yaml
    NOTES.md                ← anotaciones internas, EXCLUIDO del build público
    hitos/  hechos/
  personas/<slug>.yaml
  organizaciones/<slug>.yaml
  documentos/<slug>.yaml
  delitos/<slug>.yaml
  signals.yaml              ← bandeja del watcher
/schemas/                   ← JSON Schemas que validan los YAML
/scripts/                   ← validación, watchers, builds
/src/                       ← código Astro
  pages/
    index.astro             ← wrapper mínimo → PgHome lang="es"
    cat/                    ← rutas catalanas (vacío en MVP)
  components/
    pages/                  ← Pg* — lógica real de cada página
  layouts/
  styles/
/.agents/skills/            ← skills locales del proyecto (canónico)
/.claude/skills/            ← symlinks de compatibilidad → /.agents/skills/
/.github/workflows/         ← CI
```

## Convenciones

### Patrón `Pg*` para páginas

Las `.astro` en `/src/pages/` son **wrappers mínimos**. La lógica real vive en componentes `Pg*` en `/src/components/pages/`. Las páginas pasan `lang` como prop.

```astro
---
// src/pages/index.astro
import PgHome from '@/components/pages/PgHome.astro';
---
<PgHome lang="es" />
```

```astro
---
// src/components/pages/PgHome.astro
import BaseLayout from '@/layouts/BaseLayout.astro';
interface Props { lang?: 'es' | 'ca'; }
const { lang = 'es' } = Astro.props;

// TODO i18n: cuando se active el catalán, añadir rama 'ca'.
const strings = { /* castellano hardcoded */ };
---
<BaseLayout lang={lang} title={...}>...</BaseLayout>
```

### I18n


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidchicano/presuntamente](https://github.com/davidchicano/presuntamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
