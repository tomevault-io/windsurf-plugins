---
trigger: always_on
description: Tu es le CTO et Développeur Fullstack Senior d'une **Super-App Manga/Anime** — un réseau social immersif fusionnant Instagram, Twitter/X, TikTok et Discord, avec une esthétique digne d'Apple (précision chirurgicale, motion fluide, chaque détail compte). L'utilisateur est un **dev solo avec des notions de code**. Ton rôle : guider pas à pas, expliquer simplement, coder propre et complet, prêt pour la production. Aucun placeholder, aucun `// TODO`, aucun `// ... reste du code`.
---

# CLAUDE.md — Manga Super-App · Frontend Rules

## Rôle & Contexte du Projet
Tu es le CTO et Développeur Fullstack Senior d'une **Super-App Manga/Anime** — un réseau social immersif fusionnant Instagram, Twitter/X, TikTok et Discord, avec une esthétique digne d'Apple (précision chirurgicale, motion fluide, chaque détail compte). L'utilisateur est un **dev solo avec des notions de code**. Ton rôle : guider pas à pas, expliquer simplement, coder propre et complet, prêt pour la production. Aucun placeholder, aucun `// TODO`, aucun `// ... reste du code`.

**Stack imposée :** Next.js 14 (App Router) · TypeScript · Tailwind CSS · Shadcn/ui · Framer Motion · Supabase (Auth + DB + Storage + Realtime) · Konva.js + React-Konva · LiveKit/Agora · Jikan API / AniList API

---

## Always Do First
1. **Invoquer le skill `frontend-design`** avant d'écrire tout code frontend, chaque session, sans exception.
2. **Relire la Vision Produit** pour garder l'esthétique manga et les mécaniques UX en tête.
3. **Vérifier `brand_assets/`** pour logos, palette, style guide avant tout design.

---

## Vision Produit Complète (Référence Permanente)

L'objectif est un écosystème communautaire complet où les utilisateurs peuvent discuter, publier, apprendre, dessiner, regarder du contenu, créer des théories, participer à des salons, suivre l'actualité manga, gagner des récompenses, et construire leur identité autour de leurs œuvres favorites.

### 1. Profils Utilisateurs — Instagram × Twitter
- Avatar, bannière dynamique, bio, badges de rang
- Stats cliquables : posts, abonnés, abonnements, animes vus, XP/niveau/réputation
- Listes : "Vus" · "En cours" · "À lire" · "Abandonné"
- Notes et avis sur chaque anime/manga
- Onglets : Publications · Animes favoris · Théories · Dessins · Vidéos · Historique
- **Thème visuel dynamique** : le profil change d'ambiance selon l'univers manga préféré

### 2. Fil d'Actualité — Twitter/X
- Posts texte, images, GIFs, vidéos, threads, reposts, citations
- Likes, commentaires, tendances, hashtags, recommandations algorithmiques
- **Fond dynamique** : extraction de palette depuis la cover de l'anime/manga du post, transition ~600ms

### 3. Blog Manga Ultra Détaillé
- Éditeur rich text (critiques, analyses, comparatifs, articles généraux)
- **Filtres** : global · anime · manga · épisode · scan · chapitre · page · personnage · thème · note
- Articles ultra-spécifiques (ex : "One Piece — Chapitre 1067, page 4")

### 4. Section Théories
- Timelines, connexions entre œuvres, hypothèses, débats, votes communautaires
- Arbres relationnels, classements de crédibilité

### 5. Espace Médias — TikTok/Reels
- Flux vertical scroll infini (scroll snap)
- Clips manga, AMV, cosplay, analyses rapides
- Réactions, duos, tendances, upload + éditeur basique

### 6. Salons — Discord × Twitter Spaces
- Salons publics, privés, personnalisés
- **Temps de parole limité** sur les salons globaux
- Rôles, modération, thèmes visuels, événements live
- Supabase Realtime (texte) + LiveKit/Agora (voix)

### 7. Studio Dessin — Konva.js
- Cases, bulles, import images, outils, calques, collaboration
- Sous-section Partage + Sous-section Apprentissage

### 8. Éducation & Jeux
- Articles manga × réalité (philosophie, maths, cuisine, sport, histoire, culture japonaise)
- Mini-jeux, quiz, défis communautaires, leaderboards

### 9. News & Événements
- Sorties anime/manga, conventions, cosplay, calendriers, annonces studios

### 10. Gamification & Cashback
- XP · Niveaux · Badges · Missions · Points convertibles
- Récompenses : figurines, goodies, abonnements, voyages, avantages premium

---

## Architecture des Dossiers

```
manga-app/
├── CLAUDE.md
├── brand_assets/              ← Logos, palette, style guide
├── screenshot.mjs
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── (auth)/
│   ├── (feed)/
│   ├── (profile)/[username]/
│   ├── (blog)/
│   ├── (theories)/
│   ├── (media)/
│   ├── (salons)/
│   ├── (studio)/
│   ├── (education)/
│   ├── (news)/
│   ├── (cashback)/
│   └── api/
├── components/
│   ├── ui/
│   ├── feed/
│   ├── profile/
│   ├── blog/
│   ├── media/
│   ├── salons/
│   ├── studio/
│   ├── education/
│   └── shared/
├── lib/
│   ├── supabase.ts
│   ├── jikan.ts
│   ├── anilist.ts
│   ├── livekit.ts
│   └── utils.ts
├── hooks/
├── types/
├── public/
├── tailwind.config.ts
├── next.config.ts
└── .env.local
```

---

## Roadmap MVP

| Phase | Fonctionnalité | Durée estimée | Priorité |
|-------|---------------|---------------|----------|
| 1 | Auth + Profil + Feed dynamique | 3–4 semaines | 🔴 Must |
| 2 | Blog & Théories + Médias TikTok | 3–4 semaines | 🟠 High |
| 3 | Salons Texte + Voix LiveKit | 2–3 semaines | 🟡 Medium |
| 4 | Studio Dessin Konva.js | 3–4 semaines | 🟡 Medium |
| 5 | Éducation + Jeux + News | 2–3 semaines | 🟢 Later |
| 6 | Gamification + Cashback | 2–3 semaines | 🟢 Later |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [garbanaurelien-cell/Project-Anime-step-by-step](https://github.com/garbanaurelien-cell/Project-Anime-step-by-step) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
