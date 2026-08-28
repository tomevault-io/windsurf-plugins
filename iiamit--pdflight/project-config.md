---
trigger: always_on
description: Drop this in the repo root as `CLAUDE.md`. Put the build plan at `docs/BUILD-PLAN.md`.
---

# PDFlight - Claude Code Handoff

Drop this in the repo root as `CLAUDE.md`. Put the build plan at `docs/BUILD-PLAN.md`.

**Precedence.** CLAUDE.md is normative for schema, ids, corpus, rules, and acceptance criteria. `docs/BUILD-PLAN.md` is normative for architecture, rationale, and phases beyond the current one. Where they conflict, CLAUDE.md wins and BUILD-PLAN.md is the one to correct.

---

## Kickoff prompt

```
Read CLAUDE.md and docs/BUILD-PLAN.md, then execute Phase 1.

Do not start Phase 2 without checking in. Do not invent FAA URLs or
legal interpretation citations. Every source URL must be verified with
a live request before it enters manifest/sources.yaml.
```

---

## 1. What this is

PDFlight builds a single hyperlinked PDF containing the FAA reference corpus a pilot needs for any certificate or rating: handbooks, ACS and PTS, the AIM, full 14 CFR text, Advisory Circulars, and selected Chief Counsel legal interpretations. Every ACS element links to the regulations, handbook sections, and guidance that support it.

Free, open source, offline, self-updating. Distributed through GitHub Releases, linked from the aviation page at iamit.org.

The FAA content is public domain and commodity. The value is the crosswalk link layer and the rebuild pipeline.

**Prior art**: the VSL ACE Guide, a $97 commercial product with the same shape. We are not copying its taxonomy, menu structure, or naming. Build the arrangement from the ACS outline independently.

---

## 2. Locked decisions

| Decision | Value |
|---|---|
| Repo | `pdflight` |
| License | MIT for code, templates, crosswalk. FAA content is US Government work, public domain |
| Certificates | Private, Instrument, Commercial, ATP, CFI, CFII |
| 14 CFR | Full text, typeset in-file, offline |
| CFR parts | Title 14: 1, 43, 45, 47, 48, 61, 67, 68, 71, 73, 91, 103, 105, 119, 135. Title 49: 830 |
| Excluded parts | 97 (TERPS), 121 (~1,800 pages, out of scope) |
| Original content | None. FAA and NTSB source material only |
| Legal interpretations | 34 selected, see section 7 |
| Branding | iamit.org aviation theme, tokens in section 6 |
| Cadence | Event-driven, two-tier release policy. Never calendared |
| Output | One linearized PDF, PDF 1.7 |
| Distribution | GitHub Releases, `releases/latest/download/pdflight.pdf` |

---

## 3. Hard rules

Violating any of these breaks the product. They are not preferences.

**Sources**

1. Never invent or guess a URL. Verify with a live request before it enters the manifest.
2. **Never invent a citation. Looking one up is not inventing one.** The prohibition is on fabricating a year, addressee, or subject to make a URL resolve. Searching the Chief Counsel index for a known addressee and reading the year off the actual document is verification, and it is required. Twelve entries in section 7 carry no year and must go through discovery (section 4.4) before they can be verified. What is forbidden: trying years until one 200s, or substituting a different interpretation that looks similar. Drop what cannot be confirmed.
2a. The same rule governs `faa_number`. Every edition letter in section 5 was written from memory and none is verified. Do not treat them as authoritative. They are derived from the fetched document, never authored by hand. The field is informational and nullable, never a key. A document whose number will not extract is not an error. See 4.2 and 4.3.
3. FAA and NTSB material only. No Garmin, no manufacturer POH, no third-party study material. If it is not a US Government work, it does not ship.
4. Add no original content. No summaries, no mnemonics, no annotations. The "source material only" property is the whole point.

**PDF construction**

5. Link targets are logical refs, never page numbers. `14cfr:91.155`, `phak:ch15:airspace-class-b`, `acs:PA.I.B.K1`.
6. Simple `/GoTo` actions with named destinations only. No JavaScript, no `GoToR`, no embedded files, no form fields, no optional content groups.
7. PDF 1.7. Not 2.0. Mobile reader support for 2.0 is inconsistent.
8. Builds are deterministic. Same inputs produce a byte-identical SHA-256. Pin `SOURCE_DATE_EPOCH`, fix the PDF `/ID`, strip or pin `CreationDate` and `ModDate`, sort every directory listing. The release job depends on this to decide whether anything changed.
9. Never commit source PDFs or the output PDF. GitHub rejects files over 100 MB in-repo. Sources live in `cache/` (gitignored) and the Actions cache. Output goes to Releases.

**Style**

10. No em-dashes anywhere, including code comments and generated PDF text. Use " - " with spaces or restructure.
11. Active voice, concise. Strunk and White.

**Toolchain**

12. Any tool not available identically on Windows and `ubuntu-latest` is a determinism risk. Prefer Python wheels over external binaries. If an external binary is unavoidable, pin its version in both environments and record it in the lock file.

---

## 4. Phase 1 scope


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iiamit/pdflight](https://github.com/iiamit/pdflight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
