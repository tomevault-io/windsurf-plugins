---
trigger: always_on
description: - Tout le développement se fait directement sur `develop` ; `main` ne reçoit
---

# Guide de travail

## Branches & dépôt

- Tout le développement se fait directement sur `develop` ; `main` ne reçoit
  que les merges de release. Pas de branches de travail intermédiaires, y
  compris celles que l'outillage propose automatiquement (`claude/...`).
- Aucun commit, push, merge ou Pull Request sans demande explicite du
  mainteneur — les analyses et discussions ne modifient pas le dépôt.
- Commits sous l'identité du mainteneur (celle de l'historique du dépôt).
  L'environnement de travail est recyclé entre sessions : vérifier
  `git config user.name`, la branche courante et faire `git fetch` avant
  tout commit.
- Releases : bump de `homekit/package.json` (`npm version X.Y.Z
  --no-git-tag-version`) et changelog en anglais préparés à la demande ;
  le merge develop→main, le tag et la release GitHub sont faits par le
  mainteneur.
- Toute évolution est d'abord consignée dans une issue GitHub (en français,
  factuelle, mesures à l'appui) ; le développement ne démarre que sur
  demande explicite.

## Méthode

- **« Mesurer avant de croire »** : aucune conclusion ni optimisation sans
  mesure avant/après sur le vrai matériel. Distinguer explicitement fait
  mesuré et hypothèse. Une enquête n'est close qu'après validation terrain.
- Bug rapporté : obtenir le symptôme exact et les logs avant d'auditer le
  code ; quand la cause est ambiguë, instrumenter (logs) avant de corriger.
- Tester dans le vrai pipeline : `rpicam-vid`/`rpicam-jpeg` en standalone
  sont infidèles au live (flags ignorés — `--ev`, framerate — et mode
  capteur non-binné).
- Corrections de bugs ≠ features : rien de non-validé sur du vrai matériel
  dans les release notes ni la liste de features du README (bêta au mieux).
- Les notes durables vont dans le dépôt (issue ou doc), jamais dans un
  répertoire temporaire. L'environnement n'a pas d'accès web général :
  les documents externes (PDF, articles) sont fournis en upload.

## Cible matérielle — Pi Zero 2 W (prioritaire)

Le projet vise en priorité le **Pi Zero 2 W** ; ses contraintes s'appliquent
à toute décision :

- 4× Cortex-A53 @ 1 GHz, **512 Mo de RAM partagés avec le GPU**, pas de
  dissipateur (75-78 °C en charge).
- Encodeur H264 matériel **limité à 1920×1080 sur TOUS les Pi** — VideoCore IV
  (Zero 2 W/Pi 3) comme VideoCore VI (Pi 4). Mettre `width > 1920` fait planter
  `VIDIOC_STREAMON` (testé : OV5647 2592×1944 sur Zero 2 W, IMX708 2304×1296
  sur Pi 4). Ne jamais recommander de 2K/4K.
- Pas de calcul lourd, attention à la charge CPU et à la mémoire.
- **Ne jamais appeler `capture_array("main")` hors du thread caméra** pendant
  que l'encodeur H264 tourne : contention des buffers picamera2 → pic de charge
  (load 7+). Déjà rencontré, déjà corrigé — ne pas réintroduire.
- Pi Zero W v1 (ARMv6) : expérience close, documentée dans TROUBLESHOOTING —
  pas de support officiel, ne pas rouvrir.
- Pi 5 : chantier 2K/4K **clos** (#59, 2026-07) — décision produit (prix,
  thermique, encombrement), pas un échec technique. Acquis mesurés : pas
  d'encodeur vidéo matériel ; x265 logiciel ≈ 19,4 MP/s par cœur A76 → le
  2K HEVC est hors de portée, le 2K30 H.264 logiciel tient (~40 % du SoC) ;
  le protocole live de la nouvelle spec HKSV (tvOS 27) est intégralement
  cartographié dans l'issue (RTP/SRTP multi-palier, Setup Endpoints en
  write-response, SSRC assignés par le contrôleur, entiers TLV en longueur
  minimale). Code du spike archivé sur la branche `archive/pi5-hksv-spike`.
  Ne pas rouvrir sans nouveau matériel ou demande explicite.

## Architecture

Deux services applicatifs + un serveur RTSP, reliés par mediamtx :

- **`camera/` (Python, service `pi4cam`)** : picamera2 → pipe H264 →
  `RtspPublisher` (ffmpeg → mediamtx) ; piste lores YUV → `PresenceDetector`
  (MOG2 → webhook HTTP vers l'app Node sur motion). Entrée : `camera/main.py`.
  Aussi : `usb_camera_manager.py` (backend UVC bêta, sélectionné par
  `camera.source: usb`) et `control_server.py` (endpoint localhost:8990 —
  bitrate dynamique #47 et keyframe immédiate #43, piloté par l'app Node).
- **`homekit/` (TypeScript/HAP-NodeJS, service `pi4cam-homekit`)** : live stream
  (ffmpeg `-c:v copy` depuis RTSP), snapshot, capteur de mouvement, HKSV
  (prebuffer fMP4 + recording delegate). Entrée : `homekit/src/main.ts`.
- **mediamtx (service `mediamtx`)** : serveur RTSP local (`rtsp://localhost:8554/camera`).

Le live et HKSV sont du **passthrough H264 matériel** — aucun ré-encodage.
Le GOP (`iperiod` dans `camera_manager.py`) est à `fps × 1` (keyframe toutes
les 1 s) : compromis latence live ↔ fragments HKSV. Le live `-c:v copy` ne peut
rien afficher avant de recevoir une keyframe, donc un GOP court réduit le délai
d'apparition. Ne pas augmenter sans raison (ça rallonge le time-to-first-frame).

## Commandes

Tests (les deux suites doivent passer, c'est ce que vérifie la CI) :

```bash
pytest                              # tests Python (racine)
cd homekit && npm test             # tests TypeScript (vitest)
cd homekit && npm run build        # compile TS → dist/
```

Déploiement sur le Pi (installe dans `/opt/pi4cam`, dépendances Python 100 %
apt — pas de venv ni pip —, build le Node, fait un deep-merge de config.yaml
qui préserve les valeurs utilisateur) :

```bash
sudo bash install.sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexBtlle/pi4-IA-Homekit-Camera](https://github.com/AlexBtlle/pi4-IA-Homekit-Camera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
