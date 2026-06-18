---
trigger: always_on
description: \# Projet VERTEX — Contexte
---

\# Projet VERTEX — Contexte



Application Python/Streamlit d'analyse de performance trail running.

Entrée : fichiers GPX/TCX/FIT. Sortie : score physiologique + rapport PDF.

Positionnement : "Strava montre ce que tu as fait. VERTEX explique ce que ça t'a coûté."

Cible : élites trail mondiaux. Déployé sur Streamlit Community Cloud.



\---



\## Stack \& Architecture



\*\*Python\*\* : 3.11–3.14 (requis par scipy 1.17.1)

\*\*Dépendances critiques\*\* :

\- `streamlit` (lancement : `python -m streamlit run app.py`)

\- `pandas==2.2.3` — NE PAS upgrader (Copy-on-Write pandas 3.0 casse le code)

\- `scipy==1.17.1`

\- `plotly`

\- `numpy`

\- `weasyprint` — PDF HTML/CSS → moteur principal (Linux / Streamlit Cloud)

\- `jinja2` — template Jinja2 pour le PDF WeasyPrint

\- `reportlab>=4.0` — fallback PDF si WeasyPrint indisponible (Windows local)

\- `fpdf2` — conservé (héritage, non utilisé en prod)

\- `lxml` — parsing GPX



\*\*Architecture modulaire\*\* :

```

gpx\_parser.py   → parsing GPX, Haversine, extract\_race\_info

tcx\_parser.py   → parsing TCX

fit\_parser.py   → parsing FIT (Garmin natif)

engine.py       → GAP (Minetti 2002), cardiac\_drift, score, VERDICT

charts.py       → graphiques Plotly + générateur PDF (WeasyPrint + fallback ReportLab)

app.py          → UI Streamlit uniquement (ce fichier)

test\_engine.py  → suite de tests (sections A→S)

```



\*\*Déploiement\*\* : Streamlit Community Cloud · repo `vertex` · branche `main`

\*\*Feedback\*\* : Tally.so (https://tally.so/r/zxeJPM) → Google Sheets



\---



\## Commandes essentielles

```bash

\# Lancer l'application

python -m streamlit run app.py



\# Lancer les tests

python -m pytest test\_engine.py -v



\# Déployer : git push sur main → auto-déploiement Streamlit Cloud

```



\---



\## Conventions de code



\- \*\*Corrections chirurgicales uniquement\*\* — jamais réécrire un fichier entier sans accord explicite

\- Fonctions vectorisées numpy prioritaires sur `.apply()` pandas

\- `@st.cache\_data` sur toutes les fonctions de parsing

\- PDF : WeasyPrint + Jinja2 · fonts Barlow Condensed + DM Mono (TTF dans `assets/fonts/`) · fallback ReportLab sur Windows

\- Imports modules VERTEX : toujours en haut de `app.py`, jamais inline

\- Cadence GPX Garmin : valeur brute unilatérale → multiplier ×2 si < 110 spm



\---



\## Fonctionnalités actives



\- \*\*GAP Engine\*\* : modèle Minetti 2002, version scalaire + vectorisée

\- \*\*cardiac\_drift()\*\* CDC v1.4 : COLLAPSE A/B · NEGATIVE\_SPLIT · DRIFT-CARDIO · DRIFT-NEURO · DRIFT · STABLE

\- \*\*Score performance\*\* : zones dynamiques (FLAT/ASCENDING/DESCENDING) + pondération adaptative

\- \*\*VERDICT\*\* : matrice V1→V7 + V5-C + INSUFFICIENT + V1-NS

\- \*\*PDF\*\* : WeasyPrint + Jinja2 · design system Claude · Barlow Condensed + DM Mono · dark theme · hero score · métriques · quartiles · pattern · reco · élévation SVG · disclaimer médical · 2 pages si contenu déborde

\- \*\*Tally form\*\* : placé HORS des blocs `if st.button` (persist on rerender)

\- \*\*Couche traduction athlète\*\* : zéro terme interne visible (pas de EF, decay\_ratio, COLLAPSE A/B)



\---



\## Sprint actif

\*\*Sprint 9 — CLOS ✅\*\* · 30/04/2026
\- A1 ✅ · CNT Antony · MIXED confirmé · ef\_iso documenté · bug DESCENDING non reproductible
\- A2 ✅ · règle iso-pente 2 paliers · seuils delta\_fc provisoires · commit f1a9cb9
\- A3 ✅ · Coralie Ventoux CDF · baseline STABLE CDC-R2 #4 · commit 9508b57
\- CDC-R2 ✅ · commit 799c76f

\*\*Sprint 10 — OUVERT\*\* · 01/05/2026
\- CD-2 ✅ · PDF redesign complet · WeasyPrint + Jinja2 · design system Claude pixel-perfect · fonts Barlow/DMMono embarquées · _build_weasy_context() + _generate_weasyprint() · fallback ReportLab Windows · packages.txt GTK · élévation SVG · 2 pages · 220/220 verts · 11/05/2026
\- CD-3 · marketing deliverables · spec à écrire
\- SCR-U1 · spec à écrire
\- ELV-B1 · spec à écrire


\---



\## Bugs connus \& décisions figées



\*\*BUG-2\*\* : COLLAPSE + decay < 0.80 → doit déclencher V5-C. Corrigé dans matrice, à intégrer Sprint 5.

\*\*Faux positif V7\*\* : angle mort structurel départ montée (dataset Coralie Toureille). Comportement connu, non corrigé.

\*\*GAP dégradé < 2–3% gradient\*\* : limitation connue modèle Minetti. Disclaimer en place.

\*\*EF biaisée faible D+\*\* : SCI-1 disclaimer actif, SCI-3 étude en cours.



\*\*Décisions figées — ne jamais remettre en question\*\* :

\- `pandas==2.2.3` — version verrouillée

\- Seuil COLLAPSE A : slope < -3.0 bpm/h + chute > 10% (CDC Elena v1.4)

\- Seuil COLLAPSE B : chute > 20% sur segments plats

\- Ultra : ≥50km \*\*ET\*\* ≥4h (critère ET strict — validé Elena CDC)

\- Tally form hors bloc `if st.button`



\---



\## Règles absolues



\- Ne \*\*jamais\*\* exposer les termes internes côté athlète : `EF`, `decay\_ratio`, `COLLAPSE A/B`, `dissociation CV`

\- Ne \*\*jamais\*\* toucher au design system sans validation : `#080E14` · `#41C8E8` · `#C84850` · `#C8A84B` · Barlow Condensed + DM Mono

\- Ne \*\*jamais\*\* upgrader `pandas` au-delà de 2.2.3

\- Ne \*\*jamais\*\* proposer de réécriture complète d'un fichier sans demande explicite

\- Toujours inclure disclaimers médicaux dans UI et PDF

\- Analytics Streamlit désactivé via `.streamlit/config.toml` — ne pas réactiver



\---



\## Fichiers clés




<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonypellat-alt/Vertex](https://github.com/antonypellat-alt/Vertex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
