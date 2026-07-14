---
trigger: always_on
description: Site vitrine Astro pour Montpellier Dépannage (remorquage 24/7, entreprise familiale, plus de 30 ans d'expérience).
---

# Montpellier Dépannage — Site web

Site vitrine Astro pour Montpellier Dépannage (remorquage 24/7, entreprise familiale, plus de 30 ans d'expérience).
Stack : **Astro 4** static, vanilla TS, CSS scopé par composant. Pas de framework JS (pas de React, Vue, etc.).

## Source unique de vérité

`src/config/site.ts` exporte **tout** ce qui est partagé : nom, téléphone, email, adresse, nav, photos.
Toujours lire/modifier ces données ici, jamais en dur dans les composants.

## Arborescence

- `src/pages/index.astro` — homepage, ordre des sections : Hero → CapacityStats → ServicesGrid → CoverageBand → Certifications → AssistanceCTA
- `src/components/home/` — sections de la home (un fichier par section)
- `src/components/{TopNav,Footer,LogoMark}.astro` — chrome global
- `src/layouts/BaseLayout.astro` — head, fonts, meta
- `src/styles/{tokens.css,global.css}` — design tokens + reset

## Design tokens

Tout dans `src/styles/tokens.css` (variables CSS). Référence : `design_handoff_site_web/colors_and_type.css`.

Couleurs marque : `--md-brand-green: #2C6126`, `--md-brand-yellow: #E4E13C`, `--md-brand-black: #1A190F`.
Police d'affichage : Space Grotesk. Police de corps : DM Sans. Mono : JetBrains Mono.

**Ne pas hardcoder de couleurs ou de tailles** — toujours via `var(--…)`.

## Photos

- Source brute (PDF, DOCX, JPEG haute résolution fournis par le client) : `images/` à la racine.
- Versions servies : `public/img/` (renommées en kebab-case sémantique).
- Référencer via `photos.<key>` depuis `site.ts`, jamais en URL absolue.
- Workflow pour ajouter une photo : copier dans `public/img/`, ajouter une clé dans `photos`, l'utiliser dans le composant.
- **Ne jamais hotlinker depuis l'ancien WordPress** — historiquement on l'a fait, c'est nettoyé.

## Logo

`public/img/logo-mdp.png` (extrait du `.docx` client, oval ~1.78:1).
Le composant `LogoMark.astro` force la hauteur via inline style — le ratio largeur/hauteur de l'image est ~1.78:1, donc `size=32` donne ~32×57 px.
**Ne pas remettre de wordmark texte à côté du logo** : le nom de l'entreprise est déjà dans le visuel.

## Conventions copy

- Tout en français, ton sobre et professionnel.
- Numéros de téléphone toujours via `{site.phone.display}` et `{site.phone.href}`.
- Espaces insécables (`&nbsp;`) avant `?`, `!`, `:`, `;` et autour de `—`, et entre nombre+unité (`24&nbsp;min`).

## Animations

- `prefers-reduced-motion: reduce` doit être respecté partout (transitions désactivées, animations courtes).
- Compteurs animés via `IntersectionObserver` (cf. `CapacityStats.astro`).
- Pas de bibliothèque d'animation — CSS keyframes + JS minimal seulement.

## Section AssistanceCTA — carte qui se retourne

La carte recto reproduit le **Certificat d'Assurance** français (vert sauge, watermark "CA", titre serif, champs noirs).
Le verso met en évidence le numéro d'assistance (jaune signal).
Si on touche au design de la carte, garder la cohérence avec le visuel officiel français — référence : `public/img/carte-verte-ref.jpg`.

## Backend — Pages Functions + D1

Le site est statique mais utilise **Cloudflare Pages Functions** (dossier `functions/`) pour le formulaire de devis, le dashboard admin, et le cron.

Endpoints :
- `POST /api/devis` — création d'une demande (insert D1 + email Resend avec lien signé)
- `GET  /api/devis/:id/traiter?t=<token>` — marque comme traitée (HMAC sur l'id)
- `GET  /api/cron/check-overdue` — relance WhatsApp via Evolution API pour les devis ouverts >2h. Header `Authorization: Bearer ${CRON_SECRET}` requis. À appeler via cron-job.org ou GitHub Actions toutes les ~10 min.
- `GET  /admin` — dashboard KPIs. Auth via `?token=<ADMIN_TOKEN>` puis cookie httpOnly.
- `GET  /api/pilotage/snapshot` — agrège les indicateurs du dashboard de pilotage. Auth cookie `mdp_pilotage` ou `?token=`.

Storage : **D1** (SQLite Cloudflare). Schéma dans `schema.sql`. Tables : `devis`, `call_clicks`, `kv_cache`.

## Dashboard de pilotage (`/pilotage`)

Écran affiché en continu au bureau. `src/pages/pilotage.astro` poll `/api/pilotage/snapshot` toutes les 30 s et fait défiler **3 pages** automatiquement toutes les 15 s (flèches ←/→ pour naviguer manuellement) :

1. **Opérations** — devis, relevés de temps, plannings (source : D1 + DepanTime).
2. **Conformité & flotte** — % véhicules avec CT planifié (app Flotte), taux de conformité documentaire des dépanneurs (app Habilitation).
3. **Avis clients Google** — note moyenne réseau, note du mois courant + nb d'avis 1★, note par établissement (API Google Business Profile).

Chaque indicateur porte un statut couleur (vert/orange/rouge/gris). `snapshot.ts` agrège en parallèle 4 sources externes, chacune tolérante à l'échec (carte grise « Indisponible ») :

- **Flotte** : `GET {FLOTTE_URL}/api/pilotage-public/snapshot`, `Authorization: Bearer {FLOTTE_PILOTAGE_SECRET}`.
- **Habilitation** : `GET {HABILITATION_URL}/api/pilotage/snapshot`, header `X-Pilotage-Secret: {HABILITATION_PILOTAGE_SECRET}`.
- **Google** : API Google Business Profile (OAuth refresh token). Quota faible → résultat mis en cache 30 min dans `kv_cache` (clé `google_reviews`) ; en cas d'échec on sert le dernier cache. Logique dans `functions/_lib/google-reviews.ts`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexDiLorenzo/MtpDep](https://github.com/AlexDiLorenzo/MtpDep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
