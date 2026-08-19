---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.
See .claude/skills/ for detailed instructions on generating rustmotion scenarios.

## Règle obligatoire

Tout JSON de scénario généré doit être validé avec `rustmotion validate` avant d'être présenté à l'utilisateur. Le validateur fait deux passes : **schema** et **geometry** (détection de débordement viewport). Les deux doivent passer.

## Sécurité géométrique (viewport)

Aucun contenu textuel ne doit dépasser du device. Quatre propriétés contrôlent ce comportement :

- `style.white-space` (default `normal`, donc wrap actif) sur `text` : le texte wrap sur la largeur du parent par défaut. `white-space: "nowrap"` (ou `"pre"`) est légitime uniquement si un `max-width` fini + `font-size` raisonnable garantissent que la ligne tient. Le validateur émet `unwrappable_text_overflow` sinon. Il n'existe pas de champ `style.wrap` — c'est un vocabulaire hérité de l'ancien modèle de style, supprimé de `CssStyle`. Voir [rules/geometry-safety.md](.claude/skills/rustmotion/rules/geometry-safety.md).
- `auto_scroll` (default `true`) sur `codeblock` et `terminal` : quand le contenu dépasse la hauteur du `size`, le moteur scrolle (clip + translate) sans réduire la `font-size`. `auto_scroll: false` → `auto_scroll_disabled_overflow`.
- `style.text-autofit` (default absent) sur `text` et `gradient_text` : réduit la `font-size` jusqu'à ce que le contenu tienne dans sa boîte. À réserver au texte piloté par des données, dont on ne peut pas connaître la longueur à l'avance — pas pour compenser une mise en page qu'on peut simplement dimensionner. Le rétrécissement s'arrête à un plancher de lisibilité calibré ; si ça ne suffit pas, **la violation est toujours signalée**. Seuls ces deux composants l'implémentent : le déclarer ailleurs est inerte.
- `style.overflow` (default `visible`) sur les conteneurs : sémantique CSS. `hidden` clippe au bord du parent. Le validateur ne se plaint que si le contenu sort du **viewport**, pas d'un parent `visible`.

`marquee` et `cursor` sont exemptés (leur rôle est de bleed).

CLI :
- `rustmotion validate -f file.json` — schema + geometry
- `--fix` — auto-fix sûr : `auto_scroll: true` sur `auto_scroll_disabled_overflow`, retrait de `style.white-space` sur `unwrappable_text_overflow` (retour au wrapping), et `text-autofit: true` sur `content_overflows_box` pour `text`/`gradient_text`. Les débordements de viewport restent non corrigés : ils demandent un arbitrage de mise en page. `--fix` **refuse** d'écrire sur un scénario templaté, utilisant `include`, ou utilisant `for-each`/`use` — les index de chemin ne correspondraient plus à la source.
- `--report r.json` — rapport JSON
- `--strict-anim` — vérification frame par frame ; ajoute la détection `animated_text_overflow` (transform animé qui sort du viewport à un instant échantillonné). L'échantillonnage s'arrête à `scene.freeze_at`, puisque rien n'est rendu au-delà.
- `--strict-attrs` — promeut en erreurs les attributs inconnus (détection schéma + did-you-mean, activée par défaut en warnings)
- `--lenient` — warnings au lieu d'errors

## Encodage

- ffmpeg est auto-détecté et utilisé par défaut (10-bit H.264, meilleure qualité sur les gradients sombres)
- `--hardware-acceleration` sonde `ffmpeg -encoders` et bascule sur VideoToolbox/NVENC/QSV/AMF si la machine en offre un. Indisponible → message explicite et repli logiciel, jamais de bascule silencieuse. Le CRF n'a pas de sens sur la plupart des encodeurs matériels : le passer avec l'accélération produit un avertissement.
- `--frames a-b` rend une plage de frames en segment autonome, avec **sa** tranche d'audio (les pistes ne repartent pas de zéro). `rustmotion concat seg1.mp4 seg2.mp4 -o out.mp4` les recolle via le concat demuxer de ffmpeg. C'est la brique d'un rendu distribué.
- Sans ffmpeg, le fallback openh264 intégré encode en 8-bit
- Pour les vidéos avec des gradients sombres, recommander `--codec prores` pour une qualité maximale

## Factorisation : `components`, `for-each`, `use`

Ne duplique pas un sous-arbre. Si dix cartes ne diffèrent que par leurs données, écris-en une et itère — c'est le mode d'échec le plus fréquent de la génération, chaque copie étant une occasion de diverger.

```json
"components": { "stat_card": { "params": { "label": { "type": "string" } }, "template": { … } } },
"children": [{
  "for-each": [ { "label": "Revenue" }, { "label": "Users" } ],
  "template": { "use": "stat_card", "props": { "label": "$label" } }
}]
```

Chaque élément du `for-each` lie ses champs directement (`$label`), plus `$index` et `$item`. `params` a la forme de `config` ; omettre `default` rend le paramètre requis. La clé d'overrides est **`props`**, pas `config` — ce nom-là est réservé et serait sauté par la substitution.

`components` est local au fichier qui le déclare. On peut itérer sur un tableau venu d'une variable ; on ne peut pas instancier un composant défini dans un fichier inclus. Toute erreur — cycle, tableau manquant, composant inconnu, paramètre absent — est nommée et située. Voir [rules/templates-and-iteration.md](.claude/skills/rustmotion/rules/templates-and-iteration.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeadcodeDev/rustmotion](https://github.com/LeadcodeDev/rustmotion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
