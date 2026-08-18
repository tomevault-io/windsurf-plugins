---
trigger: always_on
description: - Dorénavant, chaque fois que c'est nécessaire (nouvelle version, fix, changement de palier/tier), prends en charge toi-même la mise à jour sur **Maven Central** et sur **Gumroad** sans attendre qu'on te le demande explicitement à chaque fois.
---

# Instructions pour ce projet (ffmpeg-kit)

- Dorénavant, chaque fois que c'est nécessaire (nouvelle version, fix, changement de palier/tier), prends en charge toi-même la mise à jour sur **Maven Central** et sur **Gumroad** sans attendre qu'on te le demande explicitement à chaque fois.
- **Audit de qualité après chaque build CI réussi** : dès qu'un build passe (Free, Basic, Full, Full GPL, toutes lignes confondues), effectue un audit de qualité complet du palier concerné — JNI (fuites, use-after-free), Java (NPE, ressources non fermées, logique SRT), sécurité (exposition AAR, workflow triggers), cohérence de version, et documentation. Affiche toujours le résultat sous forme de **tableau Markdown** avec les colonnes `Sévérité | Catégorie | Description` (valeurs de sévérité : Critique / Moyen / Mineur / OK). Signale sans attendre qu'on le demande.
- Toute publication réelle vers Maven Central (tag `v*-free`) est permanente — informe l'utilisateur avant de pousser ce genre de tag, mais procède avec ton meilleur jugement si le contexte indique que c'est attendu.

## Les lignes LTS et leurs produits Gumroad/Maven

### Ligne 6.0 LTS

| Palier | Prix | Workflow CI | Tag déclencheur | Distribution |
|---|---|---|---|---|
| Free | $0 | `build-free.yml` | `v*-free` (ex: `v6.0.1-free`) | Maven Central, `dev.ffmpegkit-maintained:ffmpeg` |
| Basic | $19 / $49 (team) | `build-basic.yml` | `v*-basic` | Gumroad `dmL2RoVC0QSkAHn9SG77aA==` (https://ffmpegkit.gumroad.com/l/iqppf) |
| Full | $29 / $75 (team) | `build.yml` | `v*` (sans suffixe, exclut `-free`/`-basic`/`-gpl`) | Gumroad `sO6O3VHxKlhjlWtN4SjnCg==` (https://ffmpegkit.gumroad.com/l/ffmpegkit-lts-android) |
| Full GPL | $39 / $99 (team) | `build-gpl.yml` | `v*-gpl` | Gumroad `S0e0mRGg2W-aD3hH60qUvQ==` (https://ffmpegkit.gumroad.com/l/bctphn) |

### Ligne 7.1 LTS

| Palier | Prix | Workflow CI | Tag déclencheur | Distribution |
|---|---|---|---|---|
| Free | $0 | `build-71-free.yml` | `v*-free71` (ex: `v7.1.5-free71`) | Maven Central, `dev.ffmpegkit-maintained:ffmpeg` |
| Basic | $19 / $49 (team) | `build-71-basic.yml` | `v*-basic71` | Gumroad `62h6MdrsmlQGwn5T_4W2DQ==` (https://ffmpegkit.gumroad.com/l/msfal) |
| Full | $29 / $75 (team) | `build-71-full.yml` | `v*-full71` | Gumroad `07IGSzVpUhfg8Fo9ejndJQ==` (https://ffmpegkit.gumroad.com/l/qnaow) |
| Full GPL | $39 / $99 (team) | `build-71-gpl.yml` | `v*-gpl71` | Gumroad `5e-7hgVcyjhJLkM-kUkXAw==` (https://ffmpegkit.gumroad.com/l/cgfhid) |

### Ligne 8.1 LTS

| Palier | Prix | Workflow CI | Tag déclencheur | Distribution |
|---|---|---|---|---|
| Free | $0 | `build-81-free.yml` | `v*-free81` | Maven Central, `dev.ffmpegkit-maintained:ffmpeg` |
| Basic | $24 | `build-81-basic.yml` | `v*-basic81` | Gumroad `JAyvuyLzfKMAnL14Fx-DHg==` (https://ffmpegkit.gumroad.com/l/nxvxzc) |
| Full | $34 / $89 (team) | `build-81-full.yml` | `v*-full81` | Gumroad `d0-0nZ-6DRF_U4FntcsqdA==` (https://ffmpegkit.gumroad.com/l/sogbka) — **ACTIF** (WhisperKit JNI implémenté et validé via [whisper-demo-android](https://github.com/ffmpegkit-maintained/whisper-demo-android)) |
| Full GPL | $44 / $119 (team) | `build-81-gpl.yml` | `v*-gpl81` | Gumroad `hu_dGzO9SBO6VNTqfGhWdA==` (https://ffmpegkit.gumroad.com/l/axqjy) — **ACTIF** (WhisperKit JNI implémenté et validé via [whisper-demo-android](https://github.com/ffmpegkit-maintained/whisper-demo-android)) |

**Pour publier une nouvelle version d'un palier :**
1. Pousser le tag correspondant (`git tag vX.Y.Z-<suffixe> && git push origin vX.Y.Z-<suffixe>`) — déclenche le build CI.
2. Pour Free : si le build + 16KB alignment passent, l'étape "Publish to Maven Central" se déclenche automatiquement (gated sur `startsWith(github.ref, 'refs/tags/')`). Rien d'autre à faire.
3. Pour Basic/Full/Full GPL : **pas d'artifact public** — voir "Sécurité : pas d'exposition publique" ci-dessous. Récupérer l'AAR final depuis `ffmpegkit-maintained/ci-cache-private` (branche `<tier>` pour 6.0, `71-<tier>` pour 7.1, `81-<tier>` pour 8.1), puis mettre à jour le fichier sur le produit Gumroad concerné : `gumroad products update <product_id> --file <chemin.aar>`. Attention : `--file` AJOUTE un fichier plutôt que de remplacer — retirer l'ancien manuellement via le dashboard si besoin (pas de primitive CLI propre pour ça).

## Sécurité : pas d'exposition publique des AAR payants

Le repo `ffmpeg-kit` est **public**. Deux pièges déjà rencontrés et corrigés (2026-06-22) :

1. **`actions/upload-artifact` sur un repo public** = téléchargeable par n'importe quel utilisateur GitHub connecté (gratuit), pas seulement les collaborateurs. Retiré des 3 workflows payants (`build.yml`, `build-basic.yml`, `build-gpl.yml`). Le palier Free garde son artifact (normal, c'est gratuit de toute façon).
2. **Une branche de checkpoint sur ce même repo public** = clonable par n'importe qui sans authentification (`git clone --branch ci-cache ...`). Les branches `ci-cache`/`ci-cache-basic`/`ci-cache-gpl` exposaient le `.aar` complet — supprimées.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ffmpegkit-maintained/ffmpeg](https://github.com/ffmpegkit-maintained/ffmpeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
