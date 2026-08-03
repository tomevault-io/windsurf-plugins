---
trigger: always_on
description: Fichier de référence automatiquement chargé par Claude Code à chaque nouvelle session.
---

# CLAUDE.md — Plat du Jour

Fichier de référence automatiquement chargé par Claude Code à chaque nouvelle session.
Contient tout ce qu'il faut savoir pour travailler sur ce projet sans perdre de contexte.

---

## Identité du projet

**Plat du Jour** est une landing page pour une application mobile suisse de découverte des menus du jour.
L'app permet aux utilisateurs de trouver les plats du jour autour d'eux (géolocalisation), de réserver,
et aux restaurateurs de publier leur menu facilement.

- **URL de production :** https://plat-du-jour.vercel.app
- **Dépôt GitHub :** https://github.com/magi22/Plat-du-jour
- **Déployé sur :** Vercel (pas Netlify — le netlify.toml a été supprimé)
- **Contact chatbot :** WhatsApp/Telegram +221785421733

---

## Stack technique

| Outil | Version | Rôle |
|---|---|---|
| React | 19 | Framework UI |
| TypeScript | — | Typage |
| Vite | — | Build tool |
| Tailwind CSS | — | Styles utilitaires |
| Framer Motion | 12 | Animations entrées/sorties/présence |
| Radix UI Accordion | 1.2 | FAQ accessible |
| Embla Carousel | 8 | Carrousel témoignages |
| Lucide React | — | Icônes |

---

## Commandes utiles

```bash
npm run dev      # Serveur local
npm run build    # Build production (toujours vérifier avant push)
```

---

## Architecture des fichiers

```
src/
├── App.tsx                          # Séquençage sections + fond animé global
├── index.css                        # Animations custom + fonds animés (classes fr)
├── main.tsx
│
├── i18n/
│   ├── translations.ts              # TOUTES les traductions FR/EN/DE/IT
│   └── LanguageContext.tsx          # Context + useLanguage() hook
│
├── animations/
│   └── variants.ts                  # fadeUp, fadeLeft, fadeRight, scaleIn, staggerContainer, defaultViewport
│
├── hooks/
│   └── useScrolled.ts               # Boolean basé sur scroll threshold
│
├── lib/
│   └── getStoreLink.ts              # Détection iOS/Android → lien store adapté
│
├── constants/
│   └── testimonials.ts              # STATS + TESTIMONIALS (données statiques)
│
├── assets/
│   ├── Mascotte@2x.png              # Mascotte principale (watermark + RestaurantSection)
│   ├── Logomascotte.svg             # Logo (Header, Footer, Hero badge)
│   ├── restaurant (1).png           # Photo RestaurantSection (locale)
│   ├── *shots_so.png (8 fichiers)   # Captures AppShowcase
│   ├── apple3.gif / apple4.png ...  # Icônes Apple Store
│   └── play3.gif / play4.png ...    # Icônes Google Play
│
├── components/
│   ├── Header.tsx
│   ├── Hero.tsx
│   ├── ProblemSolution.tsx
│   ├── HowItWorks.tsx
│   ├── Features.tsx
│   ├── AppShowcase.tsx
│   ├── RestaurantSection.tsx
│   ├── Pricing.tsx
│   ├── Testimonials.tsx
│   ├── Faq.tsx
│   ├── DownloadSection.tsx
│   ├── Footer.tsx
│   │
│   └── ui/
│       ├── LanguageSelector.tsx     # Sélecteur langue (pills horizontaux)
│       ├── ChatBot.tsx              # Widget chat flottant
│       ├── ScrollToTop.tsx          # Bouton remonter en haut
│       ├── FadeIn.tsx               # Wrapper animation réutilisable
│       ├── SectionContainer.tsx     # Wrapper max-w + padding
│       └── ImageWithFallback.tsx    # Image avec fallback
│
public/
└── favicon.svg                      # Copie de Logomascotte.svg
```

---

## Ordre des sections (App.tsx)

```
Header (fixe)
Hero                     → fond clair animé
ProblemSolution          → fond clair animé
── Fondu blanc → gris ──
HowItWorks               → bg-gray-50
── Fondu gris → blanc ──
Features                 → fond clair animé
── Ligne 1px ───────────
AppShowcase              → bg #0a0a0f (noir)
── Ligne 1px ───────────
RestaurantSection        → fond clair animé
── Fondu blanc → gris ──
Pricing                  → bg-gray-50
── Fondu gris → blanc ──
Testimonials             → fond clair animé
Faq                      → fond clair animé
── Ligne 1px ───────────
DownloadSection          → bg #0a0a0f (noir)
Footer                   → bg #070710
[overlay] ChatBot + ScrollToTop
```

**Règle de transition (ne pas modifier) :**
- `blanc ↔ noir` : `<Ligne />` (1px, pas de dégradé)
- `blanc ↔ gris` : `<Fondu />` (40px gradient)

---

## Système i18n

**4 langues :** FR · EN · DE · IT

- Toutes les chaînes sont dans `src/i18n/translations.ts`
- Hook : `const { t, lang, setLang } = useLanguage()`
- Auto-détection : `navigator.language`
- Persistance : `localStorage`
- **Ne jamais hardcoder du texte dans les composants** — tout passe par `t.section.clé`

**Exceptions (données statiques non traduites) :**
- `src/constants/testimonials.ts` : STATS + TESTIMONIALS
- `src/components/RestaurantSection.tsx` : badge `+ 1'250 CHF`
- `src/components/DownloadSection.tsx` : noms des restaurants sur la carte

**Sélecteur de langue :**
- Design : pills horizontaux inline au clic (pas de dropdown vertical — ça se coupe sur mobile)
- Fichier : `src/components/ui/LanguageSelector.tsx`

---

## Système de fond animé (index.css)

Classes CSS en **français** pour lisibilité :

### Fond clair `.fa*` (sections blanches)
```html
<div class="fa">
  <div class="fa-tinte" />   <!-- dégradés radiaux rouge + jaune, animation 12s -->
  <div class="fa-grille" />  <!-- grille 48px qui défile en diagonale, 18s -->
  <div class="fa-lueur fa-l1" />  <!-- cercle rouge haut-gauche, 10s -->

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wiicode-projects/pdj-siteweb](https://github.com/wiicode-projects/pdj-siteweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
