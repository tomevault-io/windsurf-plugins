---
trigger: always_on
description: App Electron de gesture drawing en cours de portage Android via Capacitor.
---

# Gesturo — Notes pour Claude

App Electron de gesture drawing en cours de portage Android via Capacitor.
Solo dev, non-technique côté tooling — préfère les explications step-by-step
et les commits incrémentaux.

## Stack

- **Desktop** : Electron 30, AWS S3 SDK retiré côté client
- **Mobile** : Capacitor 8 (Android scaffold présent, iOS pas encore lancé)
- **Backend** : Supabase (Auth + Postgres + Edge Functions Deno) + Cloudflare R2
- **Build** : electron-builder (DMG arm64, notarize signé sur tags `v*`),
  GitHub Actions Android (debug APK artifact, JDK 21 + android-34 SDK)
- **Admin web** : app séparée vanilla HTML/JS dans `admin-web/`, déployée
  sur Cloudflare Pages (`gesturo-admin.pages.dev`). **JAMAIS embarquée
  dans le DMG public** — séparation physique via la whitelist `files` de
  package.json. Auth = magic link Supabase, gating server-side via
  `requireAdmin()` qui lit `profiles.is_admin`.
- Pas de TS, pas de bundler, pas de lint. Vanilla JS dans le renderer.

## Layout

```
main.js                      Process Electron principal — IPC, OAuth loopback,
                             window mgmt. ~720 lignes.
preload.js                   Bridge contextIsolation → window.electronAPI
moodboard-preload.js         Bridge spécifique au webview moodboard
supabase.js                  Client Supabase (PKCE + storage adapter sur disque)
config.js                    Constantes publiques (SUPABASE_URL/KEY publishable)
                             ⚠ aucun secret ici, jamais.
whitelist.json               Emails autorisés en bêta + admin_password local
.env                         Dev only — NON inclus dans le DMG (cf. package.json
                             "files"). Gitignored. Si secrets compromis,
                             ROTATER côté Supabase function secrets.

index.html                   Squelette HTML (~470 lignes) — markup uniquement
styles/
  base.css                   reset, body, .screen toggle, noise overlays
  screens/{config,session,anim,recap,cinema}.css
  components/{favs,options,streak,history,badges}.css
src/
  app.js                     Le gros monolithe renderer (~1600 lignes).
                             State, auth init, folder loading, categories,
                             sequences, mode switching, session pose+anim,
                             recap, lightbox, favs, history, streak, grid,
                             options, badges. Tout sur le scope global.
                             Chargé avant bubbles/cinema (qui en dépendent).
  bubbles.js                 IIFE animation canvas d'accueil (~30L)
  cinema.js                  Module Cinéma — FILMS catalog + playback (~200L)
                             Dépend de logSession() exporté par app.js.

mobile/
  auth-mobile.js             Supabase PKCE + deep link com.gesturo.app://auth-callback
  mobile-shim.js             Réimplémente window.electronAPI via Capacitor
                             plugins + Edge Functions, pour que le renderer
                             desktop tourne tel quel dans la webview.
scripts/sync-web.js          Copie root → www/ et injecte les deux scripts
                             mobile/. Lancé par npm run mobile:sync.
www/                         Généré par sync-web.js — gitignored.

android/                     Capacitor Android scaffold (Manifest contient déjà
                             INTERNET + deep link auth-callback + singleTask)
capacitor.config.json

supabase/functions/
  _shared/r2.ts              S3 client R2 + requireUser(JWT) + resolveIsPro
                             (lit profiles.plan server-side) + requireAdmin
                             (lit profiles.is_admin) + helpers admin
                             (browseLevel, moveObject, deleteKeys, presignPut,
                             archiveKeyFor, unarchiveKeyFor, sanitizeFilename,
                             ADMIN_ALLOWED_ROOTS = Sessions/, Animations/).
                             r2Client a requestChecksumCalculation='WHEN_REQUIRED'
                             pour que les presigned PUT URLs marchent depuis
                             le navigateur (sinon SDK v3 bake un CRC32).
  list-r2-photos/            Auth-gated, isPro résolu côté serveur.
  list-r2-animations/        Idem. Free user → free/ uniquement.
  list-instagram-posts/      Public (cache 1h). Token IG dans secrets.
  user-data/                 favoris/sessions/streak/refreshProStatus.
                             Auth via JWT, écriture via service role.
  stripe-webhook/            (existant, pas touché récemment)
  admin-r2/                  Admin-only (requireAdmin). Multi-action via
                             body.action: 'browse' (1 niveau), 'list' (récursif),
                             'upload-urls' (presigned PUT batch), 'delete'/
                             'archive'/'unarchive' (acceptent {keys} OU
                             {prefix} pour expand récursif), 'move' vers
                             destPrefix arbitraire. Déployé avec
                             --no-verify-jwt car requireAdmin fait sa propre
                             vérif (le gateway JWT verify pose problème
                             avec les nouvelles clés sb_publishable_).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vesanerie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
