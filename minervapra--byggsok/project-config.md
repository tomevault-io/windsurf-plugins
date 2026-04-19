---
trigger: always_on
description: > Kontekst for AI-assistenter som jobber med dette repositoriet.
---

# CLAUDE.md — Byggsok

> Kontekst for AI-assistenter som jobber med dette repositoriet.
> Oppdater denne filen når prosjektet endres.

## Prosjektoversikt

**Byggsok** er en hypergraf over norsk byggesakslovgivning. Systemet kobler
tolkningsuttalelser, lovtekster, forskrifter, forarbeider, rundskriv og
sivilombudet-uttalelser til paragrafer i plan- og bygningsloven (PBL),
Byggesaksforskriften (SAK10) og Byggteknisk forskrift (TEK17).

**Repository**: `Minervapra/Byggsok`
**Hovedbranch**: `claude/claude-md-mlggpmni6yvbnv90-NmAkK` (ikke `master` eller `main`)

> **VIKTIG:** Dette repositoriet har ingen `master`/`main`-branch.
> All merge og PR-er skal bruke `claude/claude-md-mlggpmni6yvbnv90-NmAkK` som base.
> Sjekk alltid `git branch -r` og `git remote show origin` før merge-operasjoner.

### Oppstart — les disse filene først

Når du starter en ny sesjon, les alltid:

1. **`TASKS.md`** — Konsolidert oppgaveoversikt med prioriteringer,
   metrikker og status for alle aktive oppgaver. Start her.
2. **`PICKUP_PROMPT.md`** — Ferdigskrevet prompt for å starte neste
   sesjon med riktig kontekst. Brukes ved sesjonsbytte.
3. **`tasks/lessons.md`** — Lessons learned. Unngå kjente feil.
4. **`DESIGN_PRINCIPLES.md`** — Arkitekturprinsipper for kontekststyring,
   søk og informasjonsflyt. Referansedokument ved arkitekturbeslutninger.

### Avslutning — oppdater ALLTID disse filene etter utført arbeid

Etter at en oppgave er ferdigstilt og committet, oppdater ALLTID:

1. **`PICKUP_PROMPT.md`** — Oppdater med: hva som ble gjort, ny status,
   oppdaterte nøkkeltall, nye lessons learned.
2. **`TASKS.md`** — Marker oppgaven som ferdig, oppdater metrikker.
3. **`tasks/lessons.md`** — Legg til nye lessons learned hvis relevant.
4. **`research/retrieval-gap-analyse.md`** — Logg retrieval-eksperimenter
   her når du svarer på brukerspørsmål. Dokumenter hvilke dokumenter
   som ble funnet/ikke funnet, rotårsaker (R1-R6) og hvilke
   reformuleringer som traff. Se Case-malen i filen.

### Nøkkeltall

| Metrikk | Verdi |
|---------|-------|
| Noder i hypergraf | 2 329 |
| Hyperkanter | 1 460 |
| Lovparagrafer dekket | 489 (PBL 252, SAK10 104, TEK17 133) |
| Lovparagrafer med dokumenter | 264 |
| Lovparagrafer med full_text | 477 (av 489) |
| Lovparagrafer med tema-tags | 325 (66.5%) |
| Lovtekst-filer | 72 (PBL 36, SAK10 18, TEK17 18) |
| Tolkningsuttalelser | 627 (KDD 368, Sivilombudet 259) |
| Rettspraksis | 54 (HR 20, Lagmannsrett 16, RT 18) |
| Forarbeider | 7 proposisjoner |
| Rundskriv | 21 (10 gjeldende, 10 delvis oppdatert, 1 SPR) |
| Temaer | 35 |
| Strukturerte innholdsfiler | ~155 |

## Spørsmålsruting — VIKTIG

Når en bruker stiller et spørsmål om byggesak i naturlig språk:

1. **Trestegs-klassifisering — du gjør det selv.**
   Du (Claude Code) er allerede en LLM og kan klassifisere direkte.
   Ingen API-nøkkel trengs. Bruk dette workflowet (~2800-3500 tokens):

   ```python
   from byggsok.classifier import (
       build_chapter_context, parse_chapters,
       build_section_detail, read_section_texts,
       parse_classification, theme_context, parse_themes,
       classify_source_priority,
   )
   from byggsok.hypergraph_query import HypergraphQuery

   # Steg 1: Hent kapitteloversikt (~1175 tokens)
   chapter_ctx = build_chapter_context()
   themes = theme_context()

   # Les chapter_ctx og identifiser:
   #   a) 3-5 relevante kapitler: [("pbl", 1), ("pbl", 12), ("sak10", 4)]
   #   b) 1-3 relevante temaer: ["strandsone", "soknadsplikt"]

   # Steg 2: Hent berikede §§ for valgte kapitler (~500-800 tokens)
   section_detail = build_section_detail([("pbl", 1), ("pbl", 12)])

   # Les section_detail og velg 1-8 spesifikke paragrafer ELLER
   # bokstav-undernoder. For brede §§ som §20-1 og SAK10 §4-1 vises
   # bokstaver med valgbare IDer (f.eks. lov:pbl§20-1-k for
   # terrenginngrep). BRUK ALLTID bokstav-ID når spørsmålet passer
   # en spesifikk bokstav — dette gir mye bedre presisjon i søket.

   # Steg 2b: Bestem kildeprioritering
   section_ids = parse_classification('["lov:pbl§20-1-k", "lov:sak10§4-1-f"]')
   priority = classify_source_priority(section_ids)
   # priority["forskrift_primary"] = True → forskrift + veiledning er hovedkilden
   # priority["include_guidance"] = True → kun True når forskrift er primær
   # priority["include_commentary"] = True → KDD lovkommentar for PBL plandel
   # priority["max_chars"] = 20000 (forskrift-primær) / 12000 (PBL-primær)
   # priority["guidance_max_chars"] = 5000 per § (kun når primær)

   # Steg 3: Les full lovtekst for valgte §§
   # Forskrift-primær: 20k tegn med veiledning (~5k tokens)
   # PBL-primær: 12k tegn med lovkommentar (~3k tokens)
   law_text = read_section_texts(
       section_ids,
       include_guidance=priority["include_guidance"],
       include_commentary=priority["include_commentary"],
       guidance_max_chars=priority["guidance_max_chars"],
       max_chars=priority["max_chars"],
   )

   # Steg 4: Slå opp i hypergrafen med de klassifiserte §§
   theme_ids = parse_themes('["rammetillatelse", "reguleringsplan"]')
   hg = HypergraphQuery()
   results = hg.resolve_question_broad(
       section_ids, theme_ids,
       keywords=["nøkkelord", "fra", "spørsmålet"],
   )
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Minervapra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
