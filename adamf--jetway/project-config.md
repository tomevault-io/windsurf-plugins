---
trigger: always_on
description: An open-source messaging gateway for airline and GDS reservation traffic. Go,
---

# Working on Jetway

An open-source messaging gateway for airline and GDS reservation traffic. Go,
MIT, `github.com/adamf/jetway`.

## The rule that matters most: say what you actually know

Most of the formats here are defined in paid IATA publications that were not
bought. That is not a reason to guess quietly — it is a reason to be explicit
about which layer is which.

| Layer | Standing |
| --- | --- |
| `pkg/edifact` (ISO 9735), `pkg/edifact` CONTRL, `pkg/matip` (RFC 2351) | **Specified.** The documents are public. Conformance can be checked and is. |
| `pkg/typeb` limits, PDM | **Specified.** IATA's Type B whitepaper is free. |
| `pkg/padis` | **Partly.** The PNRGOV implementation guide is free and checking against it fixed four real bugs; the PNRGOV push itself is **specified**, tested against the guide's worked example verbatim. PAOREQ/PAORES remain inferred. |
| `pkg/airimp`, `pkg/avs`, `pkg/ssim` | **Inferred.** Profiles, not conformance. AIRIMP and SSIM are paywalled. The chapter 7 file layout in `pkg/ssim/file.go` is the one two independent open-source parsers implement identically and is tested against one's sample records; still a reproduction, not the manual. |
| `pkg/ndc` | **Specified.** Schemas and real carrier examples are public. |
| `pkg/mvt` | **Inferred, closely.** AHM 780/781 is paid, but OAG publishes the element tables and verbatim examples this package is built and tested against. |
| `pkg/dcs` PSM, PTM, LDM, CPM | **Inferred, closely.** RP 1715/1718 and AHM 583/587 are paid, but airports and handlers reproduce the practices' own worked examples; the parsers are tested against those verbatim. |
| `pkg/dcs` PFS, ETL | **Inferred.** No free reproduction of RP 1719/1719c was found. The layouts follow the PNL family; the category vocabulary is public. |
| `pkg/baggage` tracing | **Inferred.** WorldTracer's AHL, OHD and FWD files are the vendor's formats and are not published; the element codes and layout here are the ones handler training material reproduces, kept as an extensible profile with unknown elements verbatim. |
| `pkg/aftn` | **Specified.** ICAO Annex 10 Volume II is public; the envelope is tested against the Annex's own example. |
| `pkg/ats` | **Specified via free reproductions.** Doc 4444 is sold, but the FAA's ICAO flight planning guidance reproduces the message forms with verbatim examples the parser is tested against. |
| `pkg/acars` | **Inferred, closely.** ARINC 620 is sold; OAG publishes the element table and verbatim OOOI examples this package is built and tested against. |
| `pkg/dcs` load control | **Method specified, data representative.** The AHM 560/565 index arithmetic is the published method; `DefaultFleet` is rounded type-class data, not any operator's. |
| `pkg/fare` | **Structure inferred, data none.** Fare basis, rules, taxes and pricing follow how ATPCO filings and tickets work; the filings themselves are licensed and the package carries no fare. Callers supply a tariff. |
| `pkg/paxlst` | **Specified.** The WCO/IATA/ICAO PAXLST implementation guide is public; the package follows its segment tables and is tested against its worked examples verbatim. SSR DOCS layout is inferred from published entry formats. |
| `pkg/iatci` | **Inferred, closely.** DCQCKI/DCRCKA and their segments follow the PADIS release 01.1 structures as mirrored publicly by EDI schema vendors, element by element; the IATCI Implementation Guide (members only) was not consulted, so usage is this package's profile. |
| `pkg/inventory` | **Method specified.** Serial nested class authorisations are the textbook leg-based inventory control, EMSR-b (Belobaba; Talluri and van Ryzin ch. 2) sets them from a forecast, and leg bid prices -- additive from those ladders, or the duals of the deterministic network programme (Talluri and van Ryzin ch. 3) solved by a plain simplex -- give the network control over connecting itineraries; the numbers are the caller's. The EMSR-b tests check against the normal table, not the code. |
| `pkg/bsp` | **Specified.** IATA publishes the BSP Data Interchange Specifications Handbook (DISH 23) free; the HOT records follow its chapter 6 layouts column by column, amounts are signed by its over-punch table and add up by its section 6.7, all tested against the handbook's own figures. ADM and ACM memos carry the related document in BKS45 as chapter 6 lays it out. Net reporting, card data and tax on commission are left to bilateral schemes and not implemented. |
| `pkg/prorate` | **Method public, provisos not.** Straight rate proration by mileage is the arithmetic every prorate manual starts from; the Prorate Manual's minima, factors and special agreements are sold and not reproduced. The service charge rate is the caller's. |

When you implement something in the inferred category, say so in the package
doc, make it an extensible `Profile`, and keep unrecognised input verbatim.
Never write a doc comment that implies conformance you cannot demonstrate.

`docs/roadmap.md` has a section naming each paid document and what its absence
costs. Add to it rather than scattering another caveat: the point of collecting
them is that they are one procurement decision, not six unrelated apologies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamf/jetway](https://github.com/adamf/jetway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
