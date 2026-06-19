---
trigger: always_on
description: Structured discovery and framing workflow for website redesign projects. Guides any professional handling a paid client redesign (designer, studio owner, freelancer, consultant, in-house designer) through a 5-phase adaptive discovery process (Intake, Existing Site Analysis, Competitive Analysis, Strategic Questions, Synthesis) to produce an internal redesign framing document containing a strategic verdict, one to two key personas, a target sitemap, section plans for the homepage plus 2-3 critica
---


# audit-ux : Cadrage UX pour redesign de site web

Cette skill encode un process de cadrage pour un nouveau projet de redesign de site web client. Elle accompagne l'utilisateur : designer, studio, freelance, consultant, designer interne. En mode didactique sur toute la phase de discovery, pose les questions qu'on risque d'oublier, confronte aux angles morts, et produit en fin de parcours un **document markdown de cadrage interne** (non destiné au client) qui servira de socle pour la direction artistique et le devis.

La skill est **adaptative** : elle détecte où l'utilisateur en est dans le cadrage en fonction de ce qu'il partage, et pose les questions manquantes pour la phase concernée. Elle n'impose pas de déroulé linéaire rigide.

---

## Intégration avec une skill d'identité professionnelle (optionnel)

Cette skill est **volontairement neutre** sur les questions identitaires : elle ne prescrit pas de posture éditoriale, pas d'inspirations signature, pas de contre-mouvement créatif. Ce sont des choix qui appartiennent au professionnel qui utilise la skill, qu'il soit designer freelance, studio, agence, consultant indépendant, designer interne, ou tout autre profil.

Trois cas de figure :

1. **L'utilisateur a une skill d'identité professionnelle installée** (par exemple une skill qui encode sa posture, ses inspirations, ses partis-pris visuels, ses anti-patterns spécifiques). Dans ce cas, Claude compose les deux skills automatiquement : `audit-ux` fournit le process et les fondamentaux, la skill d'identité colore les recommandations.
2. **L'utilisateur a répondu au mini-questionnaire de calibrage à la première invocation** (voir section "Première invocation : calibrage optionnel" ci-dessous). Dans ce cas, ses réponses ont été stockées en mémoire et alimentent les recommandations.
3. **L'utilisateur n'a aucune posture déclarée**. Dans ce cas, la skill reste sur un socle neutre et produit un cadrage générique. C'est un mode parfaitement valable pour démarrer.

Dans tous les cas, la skill **nomme explicitement** sa source de posture en début de session (ex : *"Je m'appuie sur la skill d'identité `mon-studio` pour colorer mes recommandations"*, *"J'utilise les réponses de calibrage que tu m'as données précédemment"*, ou *"Tu n'as pas défini de posture, je reste sur un socle neutre"*).

---

## Première invocation : calibrage optionnel

À la **toute première invocation** de `/audit-ux` par un utilisateur, la skill détecte qu'aucune posture professionnelle n'est connue (ni skill d'identité chargée, ni réponses de calibrage en mémoire). Elle propose alors un mini-questionnaire optionnel de calibrage en 3 questions courtes.

**Logique de déclenchement** :
- Si l'utilisateur a déjà une skill d'identité chargée → pas de questionnaire, la skill compose directement
- Si l'utilisateur a déjà répondu au questionnaire dans une session précédente (réponses présentes en `userMemories`) → pas de questionnaire à nouveau, la skill réutilise les réponses
- Si rien n'est trouvé → questionnaire proposé, **explicitement skipable**

**Formulation du questionnaire** :

> Avant qu'on commence, je peux calibrer ma posture critique sur la tienne en 3 questions courtes : ou tu tapes "skip" et on démarre en posture neutre.
>
> 1. Quelle est la conviction stratégique que tu portes et que tu ne négocies pas, même si le client pousse dans l'autre sens ?
> 2. Pour quelle catégorie de clients **ne** travailles-**tu** pas : c'est-à-dire ceux que tu sais ne pas pouvoir bien servir ?
> 3. Quel est ton format de prestation principal : projet ponctuel forfaitaire, retainer/abonnement, mix, ou autre ?

**Traitement des réponses** :

- Si l'utilisateur répond aux 3 questions, ses réponses sont stockées dans `userMemories` sous forme synthétique (ex : *"Audit-ux calibrage : conviction = X, refus = Y, format principal = Z"*) pour réutilisation dans les sessions suivantes.
- Si l'utilisateur tape "skip" ou ignore le questionnaire, la skill démarre immédiatement en posture neutre. Aucun blocage.
- Si l'utilisateur répond à 1 ou 2 questions seulement, la skill utilise ce qu'il a donné et ne relance pas pour les autres.

**Règle de respect** : ce questionnaire ne s'affiche qu'**une fois par utilisateur**. Si l'utilisateur a skipé une fois, la skill ne le repropose plus dans les sessions suivantes, sauf si l'utilisateur en fait explicitement la demande (*"recalibre-moi"* ou équivalent).

**Évolutivité** : un utilisateur peut à tout moment créer une skill d'identité professionnelle séparée pour aller plus loin que ces 3 questions. Le calibrage est une porte d'entrée, pas un plafond.

---

## Inputs et outils de collecte


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FWBFstudio/ux-audit-skill](https://github.com/FWBFstudio/ux-audit-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
