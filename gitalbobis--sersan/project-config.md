---
trigger: always_on
description: > Incolla **tutto** questo file come primo messaggio in Claude Code, dalla cartella vuota del nuovo progetto.
---

  # PROMPT PER CLAUDE CODE — Ricostruzione sito SERSAN (React Three Fiber / Three.js)

> Incolla **tutto** questo file come primo messaggio in Claude Code, dalla cartella vuota del nuovo progetto.
> È scritto in italiano per te; il codice, i commenti e i contenuti del sito restano in inglese (il sito è bilingue EN/IT).

---

## 0. Ruolo e obiettivo

Sei un senior creative front-end engineer. Devi **ricostruire da zero il sito di SERSAN** (società di consulenza AI/ingegneria) con uno stack WebGL moderno, puntando alla **qualità visiva e di interazione di https://lusion.co/** — ma con un'anima adatta a una società di consulenza AI seria, regolata e premium (non un sito-studio generico e chiassoso). Deve trasmettere: rigore tecnico, credibilità enterprise, e padronanza tecnologica di frontiera.

Riferimenti di qualità (da studiare, non da copiare): lusion.co, e in generale il livello "Awwwards Site of the Day". Tagline e tono di SERSAN: **"The intelligence is artificial. The judgement stays human."**

Prima di scrivere codice, completa la **Fase 1 (setup & verifica MCP)** qui sotto e fammi un piano. Costruisci in modo incrementale e verifica visivamente ogni sezione con Playwright.

---

## 1. Stack tecnico (vincolante)

- **Build/Framework:** Vite + React 18 + **TypeScript**. (Niente Next.js: è un sito vetrina prevalentemente statico; Vite è più leggero e veloce per WebGL. Se in futuro serve SSR per SEO, valuteremo.)
- **3D / WebGL:** **three.js** + **@react-three/fiber** (R3F) + **@react-three/drei** (helper) + **@react-three/postprocessing** (Bloom, DOF, vignette, noise).
- **Animazione:** **GSAP** (timeline e sequencing) + **@gsap/react** (`useGSAP`). **Lenis** (`@studio-freight/lenis` / `lenis`) per lo smooth scroll, sincronizzato con il loop di R3F.
- **Stato:** **zustand** per lo stato globale (scroll progress, fase di caricamento, audio on/off, lingua).
- **Stile:** **Tailwind CSS** (il sito attuale lo usa già) + CSS variables per i token. Mobile-first, responsive completo.
- **Debug:** **leva** per pannelli di tuning (da rimuovere/tree-shakare in produzione).
- **Asset 3D pipeline:** modelli `.glb` ottimizzati con **gltf-transform** (Draco/Meshopt per le mesh, KTX2/Basis per le texture). Usa **gltfjsx** per trasformare i `.glb` in componenti R3F tipizzati.
- **Lingue:** EN + IT (toggle in header), come il sito attuale. Architettura i18n semplice (file di dizionari), copy EN fornito sotto, IT da generare.
- **Deploy:** **Vercel** (via Vercel MCP). Preview deploy ad ogni milestone.
- **Performance budget (obbligatorio):** 60fps su desktop recente; degrado elegante (riduci particellari/postprocessing) su mobile e su `prefers-reduced-motion`; lazy-load delle scene 3D; Lighthouse performance ≥ 80 su mobile; rispetta `prefers-reduced-motion` disattivando le animazioni non essenziali.
- **Accessibilità:** contenuti leggibili dai lettori di schermo anche con il WebGL attivo (il 3D è decorativo → `aria-hidden`), focus states, navigazione da tastiera, contrasto AA.

---

## 2. Direzione creativa & brand

**Palette (dark-first):**
- Base/background: `#0B1422` (navy quasi nero — è già il theme-color del sito).
- Superfici: gradazioni di navy/grigio-blu; testo `#F4F6FA` (off-white), testo secondario `~#8A94A6`.
- **Accento "signal"**: un gradiente luminoso usato con parsimonia per la linea animata e i momenti chiave. Proposta: **cyan elettrico → violetto** (es. `#3BE1FF → #7C5CFF`), con un secondo tono caldo opzionale per i picchi. Tienilo sobrio: monocromia navy ovunque, accento solo dove serve impatto. (Mettilo in `leva` così lo rifiniamo insieme.)

**Tipografia (mantieni quella attuale del brand):**
- **Display:** *Editorial New* (Fontshare) — serif con corsivo premium. Per gli heading grandi.
- **Body:** *Switzer* (Fontshare) — sans moderno e distintivo.
- **Mono:** *JetBrains Mono* (Google Fonts) — per eyebrow/label, numeri tabellari, micro-copy tecnico.

**Tono visivo:** tanto spazio negativo, tipografia grande e sicura, micro-interazioni precise, transizioni di pagina fluide, un layer 3D che "respira" dietro/intorno al contenuto senza coprirlo. Niente effetti gratuiti: ogni animazione deve sembrare _intenzionale e ingegnerizzata_ — coerente con un brand che dice "production-grade, non demo".

---

## 3. Effetti chiave da implementare

### 3a. La "linea colorata" che scorre con lo scroll (firma stile Lusion) — PRIORITÀ
Questo **non è un modello 3D**: è un effetto **procedurale in codice**. Implementazione consigliata:
- Definisci una **curva** `THREE.CatmullRomCurve3` che attraversa lo spazio della pagina (serpeggia tra le sezioni).
- Renderizzala come **tubo/linea con gradiente**: o `TubeGeometry` lungo la curva con uno **shader material** (gradiente animato + glow), oppure `<Line>` / `<QuadraticBezierLine>` di drei con `MeshLineMaterial`-style.
- **Guida con lo scroll:** Lenis espone il progresso di scroll (0→1). Mappalo a (a) un uniform `uProgress` che "disegna"/illumina la linea progressivamente, e/o (b) un punto luce che corre lungo la curva (`curve.getPointAt(progress)`), e/o (c) il movimento della camera lungo la curva.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GitAlboBis/Sersan](https://github.com/GitAlboBis/Sersan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
