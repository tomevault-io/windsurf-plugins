---
trigger: always_on
description: Génère des pages WordPress complètes avec Spectra (48 blocs Gutenberg) + Gutenberg core, avec ou sans Astra. Utiliser quand l'utilisateur demande de créer/générer/construire/refondre une page WordPress, une landing page, un site WordPress, ou mentionne Spectra, Astra, Gutenberg blocks, page builder, ou un template de page WP. 3 killer features cumulatives — génération depuis un brief en langage naturel, refonte intelligente d'une page existante, déploiement de templates clic-bouton (page-formati
---


# Skill `claude-skill-astra-spectra` — knowledge base v1.0-rc6

> **Ce skill n'est pas un template. C'est une base de connaissance opérationnelle pour générer des pages WordPress correctes du premier coup.** Il liste les 24 pièges Spectra documentés en production, les workflows validés, les conventions de code et les techniques natives à utiliser.

---

## ⚠️ AVERTISSEMENT À L'INSTANCE CLAUDE QUI LIT CE SKILL

Si tu es une instance Claude qui s'apprête à utiliser ce skill, **lis cette section en entier avant de faire QUOI QUE CE SOIT**. Elle contient les règles non-négociables qui empêchent les désastres documentés.

### Le skill assume du goût design + discipline visuelle. Si tu ne les as pas, RESTE STRICT.

Plusieurs sessions ont produit du markup techniquement valide (`pre-flight: STATUS OK`, roundtrip diff:0, 24 quirks verts) mais **visuellement catastrophique** parce que l'instance Claude a inventé des moves design hors baselines, n'a pas screenshot avant de claim, et a livré 5+ sections d'un coup sans validation intermédiaire.

Le user a explicitement noté la différence entre des sessions qui produisent du « niveau débutant » et celles qui tiennent la route : **la différence n'est pas le talent, c'est la discipline.**

### Règles non-négociables (anti-désastre)

#### 🔴 Règle 1 — Gate visuel BLOQUANT avant claim

Tu **ne peux pas** qualifier une composition de **« WOW / impeccable / studio editorial / propre / éditorial / beau / réussi / solide »** dans ta réponse au user **tant que tu n'as pas un screenshot validé à montrer**.

Si tu n'as pas de screenshot (tooling absent, pas d'accès agent-browser / Playwright / chrome-headless, pas de Playground actif), tu DOIS :

1. Qualifier la composition de **« non vérifiée visuellement »** dans ta réponse
2. Donner l'URL frontend au user
3. Demander explicitement : « Peux-tu me montrer un screenshot de la page rendue ? Je n'ai pas pu vérifier visuellement et je ne veux pas claim un succès sans preuve. »

Voir [`workflows/screenshot-options.md`](workflows/screenshot-options.md) pour les 5 options concrètes de capture.

#### 🔴 Règle 2 — Max 3 sections par itération, pas 8

Tu **ne livres pas** 8 sections d'un coup. Tu livres **3 sections max**, screenshot, validation user OU validation visuelle si tooling, **puis** tu ajoutes les sections suivantes.

Pourquoi : si la 1re section a un défaut design (typo trop petite, accent saturé, layout cassé), les 8 suivantes ont le même défaut. Catch tôt = corriger vite.

Exception : pour les **templates committed avec baselines screenshots** (cf `examples/` + `screenshots/`), tu peux livrer le template entier d'un coup car il a déjà été validé.

#### 🔴 Règle 3 — Référencer une baseline, ne pas inventer

Avant de poser une valeur de typographie / spacing / couleur / accent, **vérifie qu'elle est dans `references/design-baselines.md`**. Si la valeur n'y est pas, soit tu utilises la valeur recommandée par défaut du baseline, soit tu demandes au user de valider explicitement la valeur custom.

Pourquoi : « inventer 76px ou 88px en hero » sans repère = roulette russe. Le baseline donne la valeur fiable + le range acceptable.

#### 🔴 Règle 4 — Pas de 2e attempt avant que le user ait vu le 1er

Tu **ne proposes pas** « v2 » / « refonte » / « amélioration » d'une composition tant que le user n'a pas explicitement vu et commenté la v1. Sinon tu accumules du custom non-validé empilé sur du custom non-validé.

#### 🔴 Règle 5 — Pas d'images partagées entre 2 pages du même site

Tu **n'utilises PAS la même image** (même URL, même ID média) dans 2 pages différentes du même site sauf si le user l'a demandé. Réutiliser une image entre `/contact/` et `/a-propos/` cliente = signe de paresse, le user le voit immédiatement.

#### 🔴 Règle 6 — Pas plus de 3 accents couleur dans une même section

Watermark numérique géant `#FF8C00` + accent line orange `#FF8C00` + barre eyebrow `::before` orange `#FF8C00` = **3 accents identiques dans le même bloc** = saturation visuelle + perception « overdose orange » = section ratée.

Règle : 1 accent dominant par section + 1 accent secondaire max + (optionnel) 1 accent tertiaire **différent en hue** (pas la même couleur trois fois). Voir [`references/visual-pitfalls.md`](references/visual-pitfalls.md) pour les patterns à éviter.

### Anti-patterns instance Claude — ❌/✅ liste

À garder en tête tout au long de la session :

- ❌ NE JAMAIS qualifier une page de WOW / impeccable / éditorial sans screenshot
- ❌ NE JAMAIS livrer 5-6 sections d'un coup. Max 3, screenshot, valide, ajoute
- ❌ NE JAMAIS inventer typo/spacing/move design hors des baselines documentées
- ❌ NE JAMAIS proposer un 2e attempt si le user n'a pas vu le 1er
- ❌ NE JAMAIS réutiliser la MÊME image entre 2 pages du même site
- ❌ NE JAMAIS empiler 3+ accents couleur identiques dans la même section (saturation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wpformation/claude-skill-astra-spectra](https://github.com/wpformation/claude-skill-astra-spectra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
