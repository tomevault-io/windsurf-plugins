---
trigger: always_on
description: > Anciennement **inYou**. La marque est désormais **EnMoi**. Le code, les assets et la
---

# EnMoi — Application web

> Anciennement **inYou**. La marque est désormais **EnMoi**. Le code, les assets et la
> nomenclature portent encore l'ancien nom à de nombreux endroits : voir « Renommage » plus bas.

## Le produit

EnMoi est une solution de développement personnel fondée sur l'**inné** et l'**acquis**.

Le principe métier :

1. On part de la **date de naissance** d'une personne.
2. **7 formules mathématiques** (paramétrables en base) calculent 7 nombres compris entre 1 et 100.
3. Chacun de ces nombres désigne une **Force** (anciennement « aptitude ») parmi 100.
4. On génère un **PDF livrable : le PMI** (Potentiel Mental Inné), composé de pages
   d'introduction personnalisées puis de **2 pages par force** (14 pages de forces au total).

Chaque force correspond à un **rôle symbolique** fixe, déterminé par sa position dans les 7 :

| Position | Rôle symbolique |
|---|---|
| 1 | Ta colonne vertébrale |
| 2 | Ta boussole |
| 3 | Ta destination |
| 4 | Ton moteur |
| 5 | Ta vitrine |
| 6 | Ton énergie générationnelle |
| 7 | Ton inspiratrice |

### Changement majeur sur le PDF

Le contenu rédactionnel des forces **n'est plus saisi ni stocké en base**. Le client fournit
directement **2 pages PNG par force** (soit 200 fichiers). Le rôle du générateur PDF se réduit à
poser l'image PNG en fond de page et à **surimprimer deux valeurs**.

Toute la logique de mise en page textuelle des fiches (définition vivante, texte emblématique,
forces associées, zones de vigilance, mots-clés) est **obsolète** et doit disparaître.

#### Ce qui est déjà dans l'image (ne jamais le redessiner)

Constaté sur l'échantillon de 10 forces livré : les PNG sont **A4 exact à 300 DPI**
(2480 × 3508 px, ~350 Ko), déjà à la charte **enMOI**, et contiennent déjà le logo, le titre de
la force, tout le texte rédactionnel, la mention « Étape 1 » et le « (tsvp) ».

#### Ce qu'il faut surimprimer

Trois valeurs seulement, sur les zones laissées vides à dessein dans les gabarits :

| Page | Emplacement dans l'image | Valeur |
|---|---|---|
| A | bandeau turquoise, **en haut à gauche** (le logo est centré, « Étape 1 » à droite) | le **prénom** de la personne |
| B | `Force ___ /7 :` — blanc entre « Force » et « /7 » | la **position 1 à 7** |
| B | `Son rôle :` — blanc à droite du libellé | le **rôle symbolique** de la position |

⚠️ Le chiffre imprimé est la **position dans les 7**, jamais le numéro de la force (1-100). Ce
numéro n'apparaît nulle part sur les pages livrées ; il ne sert qu'à choisir le bon PNG.

Le prénom en page A est sur fond turquoise : texte **blanc**, comme les autres éléments du bandeau.

Toutes les coordonnées vivent dans **`lib/generate-pdf/overlayLayout.ts`**, et nulle part ailleurs.
Elles sont exprimées en **pixels du visuel source** (2480 × 3508), donc relevables directement dans
un éditeur d'image ; `pxToPt()` fait la conversion vers les points PDF. Quand le client réédite un
gabarit, on ajuste une constante de ce fichier et rien d'autre.

Chaque zone porte un `minFontSizePt` : la police est réduite automatiquement pour les prénoms longs
et les rôles qui déborderaient de l'espace prévu.

Méthode de calage : composer le texte sur le PNG source avec `sharp` aux mêmes coordonnées et
vérifier visuellement, plutôt que de deviner à l'aveugle.

## Périmètre et priorités

L'application finale regroupe trois parties. Ordre de travail décidé avec le client :

| Partie | Route | Priorité |
|---|---|---|
| **Back-office admin** | `/admin/*` | **Actuelle** — tout est à retravailler, fonctionnel et UI/UX |
| Accueil minimal | `/` | **Actuelle** — page simple + bouton « Se connecter / Créer un compte » en haut à droite, uniquement pour accéder au back-office |
| Site vitrine (présentation EnMoi…) | `/` et pages publiques | Plus tard |
| Espace utilisateur | `/account/*` | Plus tard |

L'accueil n'est **pas** le site vitrine pour l'instant : c'est une porte d'entrée sobre vers
l'authentification. Ne pas y investir de temps de design produit.

## Stack

- **Next.js 15.5** (App Router, Turbopack) — React 19
- **TypeScript**, alias `@/*` vers la racine
- **Tailwind CSS v4** (config CSS-first dans `app/globals.css`, pas de `tailwind.config`)
- **shadcn/ui** (`components.json`, style « new-york » — composants dans `components/ui/`)
- **Prisma 6** + **PostgreSQL sur Neon** (région `eu-central-1`, connexion pooler)
- **better-auth 1.3** (email/mot de passe, sessions en base, plugin `customSession` pour le rôle)
- **Resend** + `@react-email/components` pour les emails transactionnels
- **pdfkit** pour la génération du PMI
- **expr-eval** pour l'évaluation des formules mathématiques
- `react-hot-toast`, `lucide-react`

## Architecture

```
app/
  (web)/page.tsx              accueil public
  (admin)/admin/              back-office — layout garde le rôle ADMIN
    page.tsx                  tableau de bord (cartes de navigation)
    forces/                   médiathèque : dépôt des 2 visuels par force
    formules/                 édition des 7 formules
    pmi/                      génération du PDF
    settings/                 paramètres globaux
  (auth)/auth/                sign-in, forget-password, reset-password
  (customer)/account/         espace utilisateur (vide, plus tard)
  api/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enmoidev/enmoi](https://github.com/enmoidev/enmoi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
