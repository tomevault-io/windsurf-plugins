---
trigger: always_on
description: Jeu de casino hors-ligne pour Cardputer ADV (ESP32-S3, 240×135, 56 touches,
---

# Silicon Casino — casino de poche pour M5Stack Cardputer ADV

Jeu de casino hors-ligne pour Cardputer ADV (ESP32-S3, 240×135, 56 touches,
IMU BMI270, haut-parleur 1 W). Nom public acté : **« Silicon Casino »**
(D-036) — « Casino » en toutes lettres pour la recherche M5Burner, le
silicium pour l'identité maker. « Geek Casino » reste le nom de code du
dépôt.

## Garde-fous — non négociables sans décision explicite de Pierre

- **Jetons virtuels uniquement.** Pas d'argent réel, pas d'achat, pas de
  compte, pas de réseau (ni WiFi ni BLE). C'est un jouet et un exercice
  d'animation, pas un produit de jeu d'argent.
- Le joueur ne peut jamais être définitivement ruiné : renflouement
  automatique (voir décisions).
- Aucune reprise de matière depuis Daoa Mini (palette, design system,
  lettermark, masques, corpus). La méthode oui, la matière non.
- **Chaque jeu porte l'identité geek.** C'est une règle, pas une option :
  un jeu qui ressemble à n'importe quel casino n'a pas sa place ici.
  Deux façons de l'appliquer, selon ce que le geek touche :
  - **Par le décor** (chrome, dos de cartes, sérigraphie, cabinet) quand
    les éléments de jeu doivent rester lisibles — cartes du blackjack,
    numéros d'une roulette. Aucun réglage : c'est acquis.
  - **Par les éléments de jeu eux-mêmes** (symboles des rouleaux, faces
    de dés) quand le geek les remplace. Là, **un réglage doit permettre
    de revenir au jeu classique** : le joueur qui ne lit pas nos glyphes
    ne doit pas être exclu. Voir le réglage GLYPHS des machines à sous.

## Rôles

Pierre est product owner, directeur visuel, testeur et décideur final.
Claude Code est l'agent de développement principal — architecte,
implémenteur, opérateur du simulateur, mainteneur de la documentation.
**La transparence sur ce point fait partie du produit** (doctrine reprise
de Daoa Mini) : le dépôt public le dit ici, l'appareil le dit dans son
écran About (touche A à l'accueil), avec la distinction qui compte —
construit AVEC une IA, mais rien d'IA ne tourne SUR l'appareil.
Travail en petites étapes validées visuellement : simulateur d'abord,
appareil ensuite. Toute hypothèse douteuse se tranche par une mesure ou
un test, pas par une affirmation.

## Décisions actées

Journal complet dans [docs/DECISIONS.md](docs/DECISIONS.md). Résumé :

- Slot d'abord, **architecture multi-jeux prévue** (interface « jeu »
  commune + écran d'accueil).
- MVP : **3 rouleaux, 1 ligne** ; cœur paramétré (nb rouleaux / lignes)
  pour ouvrir 3×3 ou plus ensuite.
- **Solde persistant** (NVS) + **renflouement** en cas de ruine. Le solde
  est **partagé** entre les jeux ; la **mise** appartient au couple
  (joueur, jeu) et vit dans un bloc de sauvegarde séparé.
- **Levier IMU** (secouer/incliner) et **mode démo/attract** dès la v1.
- **RTP réaliste ~95 %** — mesuré à **95,24 %** par énumération exacte.

## Équilibrage — où vit quoi

Le **vocabulaire** (quels symboles existent) est généré depuis l'art vers
`lib/core/symbol_ids.h`. L'**équilibrage** est écrit à la main : ce sont
deux choses différentes, elles ne doivent pas se mélanger.

**Une seule table de gains** pour tous les formats — `Paytable`, dans
`paytable.cpp`, indexée `pay[symbole][nombre d'alignés]`. Une seule
fonction de calcul, `evaluateLine()`. Ne jamais recréer une table par
format : la duplication ne casse rien, elle rend le jeu incohérent en
silence (dette D-017, soldée par D-019).

Les **bandes** restent distinctes par format (`reels.cpp` pour le 3×1,
`multiline.cpp` pour le vidéo) : c'est un choix de jeu assumé, le jackpot
serait introuvable sur cinq rouleaux avec la bande du 3×1.

Toute retouche de la bande ou de la table **doit** être revalidée :

```bash
pio test -e test-native -f test_paytable
```

`exactLineRtp()` calcule le RTP **analytiquement** : la probabilité
d'aligner k symboles ne dépend que des effectifs des bandes, donc
l'énumération est inutile (elle serait de 33 millions de cas à 5
rouleaux). C'est un nombre exact, jamais une estimation. Les tests
refusent tout RTP hors de [93 %, 97 %], et deux d'entre eux comparent le
résultat au millionième à sa valeur d'avant l'unification.
Un `static_assert` casse la compilation si l'art change le nombre de
symboles : il faut alors refaire l'équilibrage, pas rallonger le tableau.

## Identité visuelle — design system

Direction actée (D-007, D-008, D-009) : **pixel-art néon**, palette
« nuit d'arcade », glyphes geek/maker, interface **anglaise**, animation
en **escalade selon le gain**. Le jackpot est le **space invader**.

Le **cabinet est une carte électronique** (trous de fixation, pistes,
vias) et le **levier est dessiné à droite**, hors cabinet : le geste IMU
actionne un objet visible. Les modules se nomment platement — **SLOTS**,
pas de nom poétique ; c'est « Silicon Casino » qui porte l'identité.

Le design system vit sur claude.ai/design, projet « Geek Casino — Design
System » — son identifiant est dans `private/design-system.md`, hors
dépôt car celui-ci est public. Il se régénère depuis le dépôt :

```
design/tokens.json        palette + géométrie — SOURCE DE VÉRITÉ
design/tools/art_*.py     glyphes 16x16 et fonte 5x7 — SOURCE DE VÉRITÉ
design/tools/gen.py       produit les cartes ET les en-têtes C++

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chatelp/geek-casino-cardputeradv](https://github.com/chatelp/geek-casino-cardputeradv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
