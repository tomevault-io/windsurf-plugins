---
trigger: always_on
description: Projekt-Kontext für AI-Sessions. Kurz, dafür Pflicht-Lektüre.
---

# CLAUDE.md — Zahnärztehaus Arch (build/)

Projekt-Kontext für AI-Sessions. Kurz, dafür Pflicht-Lektüre.

Stand: 2026-05-16.

---

## Was diese Site ist

Statische HTML-Site für die Zahnarztpraxis Zahnärztehaus Arch (Bern, BE).
Deployment: **Cloudflare Pages** mit Pages Functions für `/api/contact` und `/api/reviews`.
Kein Build-Step (kein React, kein TS-Compile) — Edits gehen direkt auf live HTML/CSS/JS.

Live-URL (Preview): `https://buildv.pages.dev/`
Produktions-URL (geplant): `https://zahnaerztehaus-arch.ch/`

### Deploy-Status (2026-05-16)

Direct-Upload via `wrangler pages deploy . --project-name=buildv --branch=main --commit-dirty=true`
aus dem `build/`-Verzeichnis. Voraussetzung: `CLOUDFLARE_API_TOKEN` und
`CLOUDFLARE_ACCOUNT_ID` (`6b127e35ea82a4fcb54c290943506af8`) als Env-Vars gesetzt.

Pages-Projekt **`buildv`** hat im Dashboard bereits konfiguriert:
- KV-Binding `KV` → Namespace `1cbd1411e7454012a75a8a31bb7b51c0` (`zaa-arch-kv`)
- Env-Vars: `RECIPIENT_EMAIL=gasserseverin1@gmail.com` (Test-Empfänger),
  `SENDER_EMAIL=onboarding@resend.dev` (Test-Modus), `RESEND_API_KEY` und
  `TURNSTILE_SECRET` als verschlüsselte Secrets

Pipeline-Verify per `curl`: GET liefert `{"error":"method_not_allowed"}`,
POST mit Honeypot `{"error":"spam"}`, POST ohne Turnstile-Token
`{"error":"turnstile_failed"}`. End-to-end Mail-Versand am 2026-05-16
verifiziert: Browser-Submit auf buildv.pages.dev → Mail in Posteingang
(`gasserseverin1@gmail.com` als Test-Empfänger).

**Wichtig zum Test-Modus:** `onboarding@resend.dev` als Absender liefert
ausschliesslich an die Mail-Adresse, mit der der Resend-Account registriert
ist. Für Produktion muss `zahnaerztehaus-arch.ch` in Resend per DKIM/SPF/DMARC
verifiziert und `SENDER_EMAIL` auf `noreply@zahnaerztehaus-arch.ch` umgestellt
werden (siehe `functions/README.md` §6).

Live-Seiten (10):
- `index.html` (Startseite)
- `praxis/index.html`
- `behandlungen/index.html` (Redirect-Stub auf `/#behandlungen`)
- `behandlungen/{notfall,prophylaxe,zahnerhalt,zahnersatz,implantat,parodontose,kinder}/index.html`

Editorial-System dokumentiert in `_workshop/experiment-schnell/system/13-editorial-layout-system.md`
und Pattern-Library in `…/14-pattern-library.md`. Beide Files sind teilweise veraltet — Code
ist seither evolviert (siehe Abschnitt „Aktuelle Design-Reality" unten).

---

## Pflicht-Step nach JEDER HTML-Bearbeitung

```
node _workshop/scripts/validate-html.js
```

Muss `✓ Alle 10 Seiten OK.` zurückgeben, bevor die Session als „erledigt" gilt.

**Warum:** Am 2026-05-10 war `index.html` mid-statement abgeschnitten. Der gesamte
inline-`<script>`-Block (Mobile-Nav-Toggle, Footer-Year, Notfall-Logik, Sticky-Bar,
Form-Submit) lief nicht mehr — ohne dass Lighthouse, visuelles Review oder GSC
das aufgegriffen hätten. Erst der User hat es manuell gemerkt („Mobile-Nav öffnet
nicht"), und der Root Cause war 100 Zeilen entfernt.

Das Validation-Script:
1. Prüft, ob jede Datei mit `</html>` endet.
2. Zählt Tag-Paare (`<html>`/`<body>`/`<main>`/`<header>`/`<footer>`/`<script>`).
3. Extrahiert jeden inline-`<script>`-Block (skipped: `src=…`, `application/ld+json`)
   und parst ihn mit Node's `vm.Script`. Fängt unterminated strings, mid-statement
   EOF, fehlende Brackets, alle JS-SyntaxErrors.

Exit-Code 0 = OK, 1 = Fehler. Bei Git-basiertem Cloudflare-Pages-Build wäre das
Script als Build-Command konfiguriert (siehe `functions/README.md`). Bei **Direct
Upload** (aktueller Deploy-Modus) greift das nicht — vor Upload manuell ausführen.

---

## Aktuelle Design-Reality (Stand 2026-05-15)

Hat sich seit dem ursprünglichen Editorial-System mehrfach weiterentwickelt.
Verbindlich ist immer der Code in `assets/main.css` und `assets/subpages.css`.

### Farben (so wie sie wirklich verwendet werden)

- **Hero-Hintergrund:** Deep-Blue `#1B5BC0` mit komplexem 4-Layer-Overlay-Gradient
  (siehe `.hero--fullbleed .hero__overlay` in main.css — diagonal + vertikal + horizontal)
- **Sections-Background:** Off-White `rgba(250,250,249,1)` (default) und
  `var(--c-primary-bg)` für `.section--alt` (helles Brand-Blau)
- **CTA-Button:** Teal `#134e4a` (Hover `#0c3531`), Text Off-White `#fafafa`, 8px radius
  — gilt für `.btn--primary` in main.css UND subpages.css (heute synchronisiert)
- **Cards mit Hervorhebung:** `.triage-path` und `.treat-card` haben jetzt 24px
  border-radius (statt früherer 8px)
- **Warm-Tan-Akzent:** `#f0e6d3` für einzelne Closing-Highlights
- **Near-Black:** `#0d0d0d` Footer

### Typo

- Display = Serif (Heading-Klasse, gross + fett)
- Body = System-Font-Stack
- H1 im Hero: `clamp(3.2rem, 9vw, 7rem)`, font-weight 600, line-height 0.95

### Buttons

`.btn--primary` ist Teal mit Off-White-Text auf **allen 10 Seiten** (`main.css` und
`subpages.css` syncron). Der weisse Hero-Override (`.hero--fullbleed .btn--primary`)
für den Header-CTA auf dem Foto-Hintergrund bleibt invertiert (Weiss-Button mit
Teal-Text), weil sonst kein Kontrast gegen das Hero-Foto.

### Editorial-Layouts (aktiv verwendet)

- `.card-grid` und `.triage-flow` (3-Spalten-Cards)
- `.editorial-split` (5/7-Grid) — wurde aus dem Form-Bereich entfernt (Form ist jetzt
  zentriert via `.section-head--centered`)
- `.intro-spread__row` — drei alternierende Reihen, vertikal überlappend, mit weissem

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GasserwerkSolutions/Zahnarzt](https://github.com/GasserwerkSolutions/Zahnarzt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
