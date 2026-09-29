---
trigger: always_on
description: **Statut de ce document.** Les sections "Objectif", "Contexte" et "Règles" sont des contraintes. La section "Design du benchmark" contient les décisions validées le 16 septembre 2026 après vérification de la doc TypeSafe et recherche de datasets. Elle remplace le premier jet. Toute modification de design se discute avant d'être codée.
---

# Jev phishing bench

**Statut de ce document.** Les sections "Objectif", "Contexte" et "Règles" sont des contraintes. La section "Design du benchmark" contient les décisions validées le 16 septembre 2026 après vérification de la doc TypeSafe et recherche de datasets. Elle remplace le premier jet. Toute modification de design se discute avant d'être codée.

## Objectif

Construire un benchmark public et reproductible de détection de phishing qui compare **Jev** (TypeSafe AI) à un **LLM classique**, puis publier les résultats sur X (@Lbdev__) avec graphiques et repo GitHub public.

Le message à démontrer avec des chiffres, pas avec des adjectifs : sur une tâche de décision de sécurité, Jev est-il assez précis, bien calibré, et combien plus rapide et moins cher qu'un LLM ? La valeur du post repose entièrement sur la crédibilité de la méthode : chaque chiffre doit être reproductible et chaque limite assumée.

## Contexte : Jev / TypeSafe AI

- TypeSafe AI a lancé Jev le 15 septembre 2026, en early access. L'accès au compte est actif.
- Jev ne génère pas de texte. Il reçoit un `state` (texte ou JSON) et des questions typées, et renvoie des réponses typées avec probabilités calibrées.
- Trois primitives, mélangeables dans un même appel, évaluées en parallèle et indépendamment :
  - `choice` : choisir une option parmi une liste (max 255 options). Renvoie `choice`, `probabilities`, `confidence`.
  - `score` : noter sur une échelle ordonnée. Renvoie `score`, `legend`, `probabilities`, `confidence`.
  - `noul` : probabilité qu'une affirmation soit vraie. Renvoie `noul` (0 à 1), sans `confidence`.
- Limites documentées : budget d'environ 32 000 tokens par requête (state et questions compris), soit environ 150 000 caractères. Le `state` est une string, un objet ou un tableau JSON : texte seulement. Service hébergé aux États-Unis (source tierce, la doc ne le précise pas).
- Prix : 42 $ par milliard de tokens d'entrée (page d'accueil typesafe.ai), soit 0,042 $ par million. Aucun prix de sortie n'est publié. Vérifié le 17 septembre 2026 sur le dashboard TypeSafe : 4 561 792 tokens consommés (3 661 776 en entrée, 900 016 en sortie, exactement la somme de nos deux passes) facturés 0,15 $, ce qui correspond à l'entrée seule (0,154 $). Le benchmark facture donc l'entrée seule, à juste titre.
- `confidence` d'un choice est dérivée de `probabilities`. Sur un choice à deux options, c'est une fonction déterministe de la probabilité max : ne pas la présenter comme un signal supplémentaire.
- Seul test indépendant publié (Every, Mike Taylor) : 0,35 s contre 8,83 s par passage face à Claude Fable 5.1, environ 25x plus rapide et 580x moins cher, mais 6 défauts détectés sur 7 contre 7 sur 7. Sur le dashboard TypeSafe, score agrégé 67,8 %, à égalité avec Sonnet 5. Attention : la référence de ces évals est la moyenne des réponses de GPT-6 Astra et Claude Fable 5.1, donc elles mesurent un accord entre modèles, pas une exactitude. Personne n'a publié d'audit de calibration indépendant : c'est l'angle du projet.

### API (vérifiée contre https://docs.typesafe.ai/api.md le 16 septembre 2026)

```
POST https://api.typesafe.ai/v1/systemone
Authorization: Bearer $TYPESAFE_API_KEY
Content-Type: application/json

{
  "state": {"from": "...", "subject": "...", "body": "..."},
  "model": "jev-latest",
  "questions": {
    "verdict": {"type": "choice", "instructions": "...", "criteria": {"phishing": "...", "legitimate": "..."}},
    "urgency": {"type": "noul", "instructions": "...", "criteria": {"true": "...", "false": "..."}}
  }
}
```

- Réponse : `model`, `answers.<id>` (mêmes clés que les questions, chaque réponse porte `type`), `usage.input_tokens` et `usage.output_tokens`.
- `criteria` est obligatoire pour choice (map option vers description ou `null`) et score (liste ordonnée), optionnel pour noul (`{"true": ..., "false": ...}`).
- Les IDs de questions ne sont pas envoyés au modèle : écrire la question complète dans `instructions`. Nommer les champs du state avec des chemins entre backticks, par exemple `` `link_url` ``.
- Erreurs : 401 clé invalide, 422 requête invalide, 429 rate limit, 529 surcharge. Retry avec backoff exponentiel sur 429 et 529.
- Docs : https://docs.typesafe.ai (index pour agents : https://docs.typesafe.ai/llms.txt). Skill installé : plugin `typesafe@typesafe-ai`.
- SDK dispos (Python `typesafe-sdk`, JS `@typesafe-ai/sdk`) mais le projet utilise l'API HTTP directe pour mesurer la latence sans couche intermédiaire et ne dépendre d'aucune signature non vérifiée.

## Design du benchmark (décisions validées)

### Données
- Dataset principal : **PhishNChips v5.2**, Hugging Face `AreLit/PhishNChips`, publié en avril 2026. 2 000 emails, 1 000 phishing et 1 000 légitimes. Corps générés par LLM autour d'URLs réelles (PhishTank, OpenPhish, GitHub Phishing DB pour le phishing, domaines Tranco pour le légitime). Licence MIT pour le contenu synthétique, sources tierces attribuées dans `SOURCE_LICENSES.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anisselbd/jev-phishing-bench](https://github.com/anisselbd/jev-phishing-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
