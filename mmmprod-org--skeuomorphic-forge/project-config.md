---
trigger: always_on
description: This skill should be used when the user asks to build physically-realistic skeuomorphic UI with Tailwind CSS — buttons, panels, gauges, knobs, CRT/LED displays, glass/metal effects, particle systems, industrial hardware. Triggers on requests mentioning: skeuomorphic, realistic depth, industrial UI, 3D button, gauge, meter, analog, tactile, material texture, retro-industrial, aerospace panel, DSP cockpit, VU meter, rotary knob, CRT display, rim light, metal recess, faceplate. Provides shadow stac
---


# SKEUOMORPHIC FORGE — Instructions de generation UI

Ce skill produit de l'UI skeuomorphique physiquement realiste. Chaque composant genere DOIT ressembler a un objet physique reel (metal usine, verre CRT, aluminium brosse, Bakelite), pas un div plat avec une ombre CSS.

Generer du code React/JSX avec Tailwind CSS + inline styles (`style={{}}`). Pas de classes CSS custom sauf indication contraire.

---

## CONTRAINTES PHYSIQUES ABSOLUES (s'appliquent a CHAQUE composant)

Violer une seule de ces regles produit un composant visuellement casse.

**C1 Ombres portees = NOIR UNIQUEMENT.** Tous les `box-shadow` non-inset utilisent exclusivement `rgba(0,0,0,...)`. Une ombre est l'absence de lumiere. Jamais d'ombre portee violette, ambree, bleue. Exceptions : (a) les `inset` highlights avec rgba clair/colore sont OK (lumiere frappant les bords internes), (b) les glows d'emission/backlight a tres faible opacite (< 0.08) sont OK pour simuler la lumiere emise par un ecran/LED (ex: `0 0 60px rgba(255,176,0,0.06)`).

**C2 Source lumineuse unique a 135deg**, coherente sur TOUS les composants de la page. Plusieurs directions = physiquement impossible.

**C3 Highlights chauds au-dessus de 0.10 d'opacite.** Tout `rgba(255,255,255,X)` avec X > 0.10 doit devenir `rgba(255,240,220,X)` ou plus chaud. Blanc pur au-dessus de 0.10 = blafard/clinique. Les edge catches a 0.03-0.08 peuvent rester en blanc pur.

**C4 Separation de contraste.** Les fonds de conteneurs/wells (#08-#10) DOIVENT etre plus sombres que les fonds de cartes (#1c-#28). Delta minimum #12 hex. Renforcer les bords de carte : `borderTop: "1px solid rgba(255,255,255,0.09)"`, `borderLeft: "1px solid rgba(255,255,255,0.06)"`.

**C5 Ombre et lumiere sont des systemes SEPARES.** L'ombre (rgba sombre) cree profondeur/recession via `box-shadow`. La lumiere (rgba chaud, gradients, pseudo-elements) cree reflets speculaires/edge catches. Ne jamais les confondre dans un stack brouille.

**C6 Minimum de couches d'ombre.** Standard (boutons/cartes/toggles) >= 5. Advanced (knobs/dials/meters) >= 8. Hero (panneaux/chassis) >= 11. Ultra (CRT vitrine) = 31. LES COMPTER apres collage.

**C7 Ne jamais inventer de shadow stacks.** Toujours copier-adapter depuis les stacks canoniques ci-dessous.

**C8 Ordre d'assemblage : arriere vers avant.** Backplate -> sous-panneaux/bezels -> wells/recesses -> instruments/displays -> hardware (vis/rivets) -> labels -> verre/reflet. Du hardware sur une surface plate = contradiction. Construire la profondeur D'ABORD.

**C9 Taille physique fixe.** Les cartes-appareils (CRT, gauge, instrument) ont des dimensions FIXES. Toutes les instances du meme composant = meme width+height. Etat disabled = memes dimensions que actif. Le contenu se centre dans le chassis. Utiliser `width` + `height` explicites.

**C10 Max 2 couleurs d'accent par page.** Un 3e accent necessite approbation explicite.

---

## VERIFICATION PRE-LIVRAISON (checker AVANT de livrer)

Un composant qui echoue un gate CRITICAL ne doit PAS etre livre.

**U0 Context Scan (BLOCKING):** La page existante a ete analysee avant de styler. Palette, boutons existants, hierarchie des conteneurs identifies. Meme role = meme style que les siblings. Shadow stack source depuis les canoniques, pas invente.

**U1 Shadow Depth (CRITICAL):** Nombre de couches respecte le minimum du tier (C6). Ombres portees NOIR uniquement (C1). Progression graduee du blur.

**U2 Light Source (CRITICAL):** Direction unique 135deg partout (C2). Lumiere/ombre separees (C5). Pas de blanc pur au-dessus de 0.10 (C3). Gradient de surface present.

**U3 Construction (HIGH):** Objet physique nomme explicitement. 4 couches : chassis + profondeur + eclairage + detail. Ordre d'assemblage respecte (C8).

**U4 Hardware (HIGH):** Vis aux coins avec radial-gradient sphere + 5 couches d'ombre + slot torx/phillips. Vis sur METAL uniquement, jamais sur verre/ecran.

**U5 Interaction States (HIGH):** hover (lift + expand shadow), active (depress + compress), disabled (opacity 0.5 + desaturate), focus-visible (outline 2px). Le shadow stack CHANGE entre les etats.

**U6 Typography (HIGH):** Body >= 13px, titres >= 14px, labels >= 11px, rien sous 10px. Texte primaire opacity >= 0.85, secondaire >= 0.5, tertiaire >= 0.35. Labels en silkscreen (text-shadow) ou grave (clip + gradient), pas en texte brut.

**U7 Accessibility (MEDIUM):** Contraste WCAG OK. `focus-visible` sur tous les interactifs. Touch targets >= 44px. `prefers-reduced-motion`. `pointer-events: none` sur les overlays de texture. Pas de `filter: blur()` dans les animations.

---

## SHADOW STACKS CANONIQUES (copier-adapter, ne jamais inventer)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MMMProd-Org/skeuomorphic-forge](https://github.com/MMMProd-Org/skeuomorphic-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
