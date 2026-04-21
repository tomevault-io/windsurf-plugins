---
trigger: always_on
description: LibMaths est un site de cours et d'annales de concours pour etudiants en CPGE (classes preparatoires). Le public cible a un **niveau faible a moyen** — les corrections doivent etre accessibles, pas ecrites "par des profs pour des profs".
---

# LibMaths — Consignes pour les corrections pedagogiques

## Contexte du projet

LibMaths est un site de cours et d'annales de concours pour etudiants en CPGE (classes preparatoires). Le public cible a un **niveau faible a moyen** — les corrections doivent etre accessibles, pas ecrites "par des profs pour des profs".

## Philosophie des corrections

### Probleme a resoudre

Les corrections classiques supposent que l'etudiant connait deja le cours, voit les astuces, et comprend pourquoi on fait telle manipulation. Pour un niveau faible, c'est comme lire la solution sans comprendre la demarche.

### Principe fondamental

Une correction classique dit **quoi faire**. Nos corrections expliquent **pourquoi on le fait**, **comment y penser**, et **ou les autres se trompent**. L'objectif n'est pas juste de comprendre un exercice, mais d'acquerir les reflexes pour les exercices similaires.

### Les 3 couches de lecture

Chaque question doit etre traitee en 3 couches :

1. **"Qu'est-ce qu'on me demande ?"** — Reformuler l'enonce en francais simple. Pas de jargon inutile.
2. **"Pourquoi on fait ca ?"** — La motivation derriere chaque etape. Pas juste "on prend le rotationnel", mais "on veut eliminer B pour obtenir une equation sur E seul — c'est comme resoudre un systeme de 2 equations a 2 inconnues".
3. **"Comment on fait ?"** — Le calcul etape par etape, avec les resultats encadres.

### Les 5 types d'encadres

Chaque correction utilise 5 types d'encadres visuels distincts :

| Type | Couleur | Classe CSS | Role | Quand l'utiliser |
|------|---------|-----------|------|-----------------|
| Rappel de cours | Bleu | `.callout--rappel` | Reposer les fondations avant d'attaquer | Avant chaque question qui necessite un theoreme ou une definition |
| "Pourquoi ?" | Orange | `.callout--pourquoi` | L'intuition, l'idee directrice, l'analogie | Avant ou pendant une etape non evidente |
| Resultat | Dore | `.callout--resultat` | La formule finale encadree | A la fin de chaque question, le resultat a retenir |
| Piege | Rouge | `.callout--piege` | L'erreur frequente au concours | Quand le rapport de jury signale une erreur recurrente |
| Methode | Vert | `.callout--methode` | La recette reutilisable | Quand la technique est transposable a d'autres exercices |

### Regles de redaction

1. **Exemples concrets avant la theorie** — Toujours montrer un tableau de valeurs numeriques ou un cas particulier avant la formule generale. Ex : montrer X=1,2,3,4 -> Y=1,1,2,2 avant d'ecrire P(Y=k).

2. **Analogies physiques** — Utiliser des images du quotidien. "Comme une balle qui rebondit sur un mur" pour la pression de radiation. "Comme lancer une piece truquee" pour la loi geometrique.

3. **Progression par difficulte** — Utiliser les badges Standard (vert) / Intermediaire (orange) / Avance (rouge). Un niveau faible doit maitriser les parties Standard, tenter les Intermediaire, ne pas paniquer sur les Avance.

4. **Chaque etape justifiee** — Jamais de "il est clair que", "on verifie aisement", "il suffit de". Si c'est une etape, on la montre. Si c'est long, on la met dans un toggle `<details>`.

5. **Les pieges du jury** — Les rapports de jury disent exactement ou les candidats se trompent. Signaler ces erreurs dans des encadres rouges "Piege classique".

6. **Toggles pour les demonstrations longues** — Les preuves optionnelles ou les verifications sont dans des blocs `<details>` pour ne pas surcharger. Titre du toggle : "Voir la preuve de..." ou "Verification detaillee".

7. **LaTeX soigne** — Utiliser KaTeX. Formules inline `$...$` pour le texte, formules display `$$...$$` pour les resultats importants. Les resultats finaux sont encadres avec `\boxed{}`.

## Structure technique

### Systeme d'annales (3 couches)

1. **`data/exercices.json`** — Index leger. Chaque reference : banque (b), epreuve (e), partie (p), questions (q), difficulte (d).
2. **`data/corrections/*.json`** — 50 fichiers, un par chapitre. Corrections question par question en LaTeX.
3. **`data/enonces/*.json`** ou `cours/annales/*.html` — Enonces structures avec ancres par question.

### Banques de concours

| Code | Nom complet | Slug URL | Couleur |
|------|------------|----------|---------|
| C | CCINP | ccinp | Bleu #4A90D9 |
| CS | Centrale-Supelec | centrale | Orange #D4873B |
| M | Mines-Ponts | mines | Vert #4CAF50 |
| X | X-ENS | xens | Violet #9C27B0 |
| E | E3A-Polytech | e3a | Rose #E91E63 |

### Format d'une reference dans exercices.json

```json
{"b": "E", "e": "Maths 1 2024", "p": "Exercice 1", "q": "3 q. : Q1, Q2, Q3", "d": "S"}
```

### Format d'un bloc de correction

```json
{
  "banque": "E3A-Polytech",
  "epreuve": "Maths 1 2024",
  "partie": "Exercice 1",
  "questions": "3 q. : Q1, Q2, Q3",
  "difficulte": "Standard",
  "intitule": "Description + ce qu'il faut savoir faire",
  "corrections": [
    {"question": "Q1", "correction": "Texte LaTeX detaille..."}
  ]
}
```

### Pages de correction standalone


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atimoz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
