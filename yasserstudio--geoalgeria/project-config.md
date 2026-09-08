---
trigger: always_on
description: GeoAlgeria is a small **pnpm monorepo** of open Algeria datasets published to npm.
---

# Working in this repo (humans & agents)

GeoAlgeria is a small **pnpm monorepo** of open Algeria datasets published to npm.
This file is the short version of how work flows here; deeper docs are linked.

## Domain vocabulary

Before writing code, data, copy, package descriptions, or issue/PR titles, read
[`CONTEXT.md`](CONTEXT.md), the project's canonical glossary, and use its terms,
avoiding the synonyms each lists under `_Avoid_`. If a concept is missing or a
term conflicts, update `CONTEXT.md` rather than drifting to a synonym (that's the
`/domain-modeling` skill).

Text folding has its own canonical terms under
[Search and normalization](CONTEXT.md#search-and-normalization): Search key,
Conservative key, Loose key, Rule, Golden corpus.

## Layout

| Path | Package | Contents |
| --- | --- | --- |
| `packages/schema/` | `@geoalgeria/schema` | shared v2 contract – types, validator, canonical metadata/manifest builders, emit helpers; a dev dependency of every generator, not itself a dataset |
| `packages/normalize/` | `@geoalgeria/normalize` | Search key generation, the Conservative key, the Loose key, the tokenizer and their Golden corpus; code only, zero dependencies, published so every consumer folds a name the same way |
| `packages/dataset/` | `geoalgeria` | wilayas, dairas, communes (+ mirrored postal data) |
| `packages/poste/` | `@geoalgeria/poste` | post offices & ATMs (Algérie Poste) |
| `packages/emploi/` | `@geoalgeria/emploi` | employment agencies (ANEM: AWEM + ALEM) |
| `packages/mobilis/` | `@geoalgeria/mobilis` | Mobilis agencies & approved points of sale (mobilis.dz) |
| `packages/telecom/` | `@geoalgeria/telecom` | cross-operator 5G coverage (Djezzy, Mobilis, Ooredoo) |
| `packages/aviation/` | `@geoalgeria/aviation` | civil airports with ICAO codes (ANAC) |
| `packages/banques/` | `@geoalgeria/banques` | licensed banks, institutions & branches (RIB/SWIFT) |
| `packages/livraison/` | `@geoalgeria/livraison` | delivery carriers & geocoded stop-desks |
| `packages/jeunesse/` | `@geoalgeria/jeunesse` | youth establishments (Ministry of Youth and Sports) |
| `packages/sports/` | `@geoalgeria/sports` | sports facilities – stadiums, pools, courts, tracks (Ministry of Youth and Sports) |
| `packages/enseignement-superieur/` | `@geoalgeria/enseignement-superieur` | higher-education network – universities, grandes écoles, ENS, centres + private & other-ministry institutions (MESRS) |
| `packages/tourisme/` | `@geoalgeria/tourisme` | tourism infrastructure – hotels, attractions, historic sites, thermal springs, parks (ASAL, OSM, Wikidata) |
| `packages/formation-professionnelle/` | `@geoalgeria/formation-professionnelle` | vocational training – CFPA, INSFP, DFEPs, private centers (MFEP / takwin.dz) |
| `packages/djezzy/` | `@geoalgeria/djezzy` | Djezzy boutiques – geocoded retail stores with category & hours (djezzy.dz) |
| `packages/ooredoo/` | `@geoalgeria/ooredoo` | Ooredoo stores – 572 EO/CSO/ESO with real coordinates & wilaya/commune linkage (ooredoo.dz locator API); completes the telecom retail trio |
| `packages/mosquees/` | `@geoalgeria/mosquees` | mosques – Wikidata + OpenStreetMap composite, bilingual, all 69 wilayas |
| `packages/sante/` | `@geoalgeria/sante` | public health establishments – EPH, EPSP, EHS, CHU (Ministry of Health), bilingual, geocoded via OSM + Wikidata |
| `packages/cliniques/` | `@geoalgeria/cliniques` | clinics & proximity-care facilities – 1,913 polycliniques, salles de soins, centres de santé, maternités & private clinics from OpenStreetMap, classified by type, bilingual, 66 wilayas; the community tier of the health sector, disjoint from `sante` (registry tier) |
| `packages/protection-civile/` | `@geoalgeria/protection-civile` | civil protection (fire & rescue) units – 880 DGPC units nationwide, Arabic-named, address/phone/fax, status tier, geocoded, official-primary (dgpc.dz), post-2026-reform wilaya linkage |
| `packages/culture/` | `@geoalgeria/culture` | cultural atlas – protected sites, museums, theatres, libraries + cultural establishments (Ministry of Culture), bilingual, fully geocoded |
| `packages/agriculture/` | `@geoalgeria/agriculture` | agriculture-sector institutions – services directorates (DSA), forest conservations, research/training institutes, chambers of agriculture, public offices & groups (Ministry of Agriculture), bilingual, geocoded |
| `packages/industrie-pharmaceutique/` | `@geoalgeria/industrie-pharmaceutique` | approved pharmaceutical manufacturers – 171 medicine (PP) & medical-device (DM) makers from the Ministry of Pharmaceutical Industry (MIP) fabrication register, bilingual, typed by nature, geocoded to commune/wilaya centroid |
| `packages/pharmacies/` | `@geoalgeria/pharmacies` | pharmacies (officines) – 3,807 geocoded across 67 wilayas, bilingual where named, phone/hours/dispensing where tagged, wilaya/commune-linked (OpenStreetMap, ODbL); honest ~half coverage |
| `packages/pharma/` | `@geoalgeria/pharma` | pharma umbrella – re-exports industrie-pharmaceutique + pharmacies in one install |
| `packages/ecoles/` | `@geoalgeria/ecoles` | schools – 11,858 primaires/CEM/lycées/préscolaires classified by cycle, bilingual, all 69 wilayas (OpenStreetMap, ODbL) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yasserstudio/geoalgeria](https://github.com/yasserstudio/geoalgeria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
