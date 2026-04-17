---
trigger: always_on
description: | Composant | Version | Description |
---

# Instructions Développement Le Hiboo - App Flutter

## Stack Technique

| Composant | Version | Description |
|-----------|---------|-------------|
| **Framework** | Flutter 3.x | Framework mobile cross-platform |
| **Langage** | Dart | Langage de programmation |
| **Architecture** | Clean Architecture | Separation of concerns |
| **State** | Riverpod | Gestion d'état réactive |
| **API** | Dio | Client HTTP |

---

## CRITIQUE - Identifiants Event (UUID vs ID numérique)

**Le backend Laravel utilise `uuid` comme identifiant pour les routes API.**

### Règle obligatoire :

Quand on mappe un `EventDto` vers une entité `Event`, toujours utiliser l'UUID :

```dart
// ❌ FAUX - utilise le hash numérique
id: dto.id.toString(),

// ✅ CORRECT - utilise l'UUID avec fallback
id: dto.uuid ?? dto.id.toString(),
```

### Pourquoi ?

- `EventDto.id` (int) : Hash numérique généré par `_parseEventId()` - utilisé pour les comparaisons internes
- `EventDto.uuid` (String) : UUID réel de l'API - **requis pour les appels API**
- Les routes Laravel comme `/favorites/{event}/toggle` attendent l'UUID via `getRouteKeyName()`

### Symptôme du bug :

```
POST /api/v1/me/favorites/1032995507/toggle
→ 404 Not Found: "Resource not found."
```

L'app envoyait un hash numérique au lieu de l'UUID (format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`).

---

## Structure Projet

```
lib/
├── core/                         # Configuration, DI, routing
│   ├── config/
│   ├── di/
│   └── routing/
├── features/                     # Features par domaine
│   ├── auth/
│   ├── events/
│   │   ├── data/
│   │   │   ├── mappers/         # EventMapper (conversion DTO → Entity)
│   │   │   ├── models/          # EventDto (modèles API)
│   │   │   └── repositories/
│   │   ├── domain/
│   │   │   ├── entities/        # Event (entité métier)
│   │   │   └── repositories/
│   │   └── presentation/
│   ├── favorites/
│   ├── home/
│   └── ...
└── shared/                       # Composants partagés
```

---

## API Backend

| Service | URL Dev | URL Prod |
|---------|---------|----------|
| API | http://api.lehiboo.localhost | https://api.lehiboo.com |

### Setup local important

**Le mobile charge `.env.development` par défaut en local** via `lib/main.dart`.
Ne pas partir du principe que `.env` est utilisé. En cas de doute :

1. vérifier `lib/main.dart`
2. modifier `.env.development`
3. relancer `flutter clean && flutter pub get && flutter run --dart-define=ENV=development`

### URLs locales selon le device

`api.lehiboo.localhost` n'est **pas** une URL universelle.

| Contexte | URL API recommandée |
|----------|---------------------|
| Navigateur local / iOS Simulator | `http://127.0.0.1:8010/api/v1` |
| Android Emulator | `http://10.0.2.2:8010/api/v1` |
| Appareil physique | `http://<IP_LAN_DU_MAC>:8010/api/v1` |

**Règle importante:** ne pas utiliser `api.lehiboo.localhost` sur appareil physique.
Sur un téléphone, `localhost` pointe vers le téléphone lui-même, pas vers le Mac du développeur.

### Quand `api.lehiboo.localhost` peut marcher

Ce domaine local ne fonctionne que si les **2 conditions** suivantes sont réunies :

1. `hosts` local correctement configuré
2. reverse proxy local sur `80/443` qui redirige vers le backend réel

Exemple d'entrées `/etc/hosts` :

```txt
127.0.0.1 lehiboo.localhost
127.0.0.1 api.lehiboo.localhost
127.0.0.1 petitboo.lehiboo.localhost
127.0.0.1 storage.lehiboo.localhost
127.0.0.1 minio.lehiboo.localhost
127.0.0.1 mailpit.lehiboo.localhost
```

Exemple Nginx :

```nginx
server {
    listen 80;
    server_name api.lehiboo.localhost;

    location / {
        proxy_pass http://127.0.0.1:8010;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Diagnostic obligatoire pour Claude avant de conclure à un bug app

Si un dev dit "l'API locale ne marche pas", **Claude doit vérifier dans cet ordre** :

1. Le backend tourne-t-il vraiment ?

```bash
docker ps | grep lehiboo-api
```

On doit voir un mapping du type `8010->8000`.

2. Le backend répond-il directement sur le port publié ?

```bash
curl -I http://127.0.0.1:8010
curl -I http://127.0.0.1:8010/api/v1/events
```

3. Le domaine local répond-il ?

```bash
curl -I http://api.lehiboo.localhost/api/v1/events
```

Interprétation :

- si `127.0.0.1:8010` répond mais pas `api.lehiboo.localhost`, le problème est le proxy/vhost local
- si rien ne répond sur `127.0.0.1:8010`, le problème est côté backend/docker local
- si ça marche sur Mac mais pas sur téléphone, le problème est l'URL choisie pour le device

### HTTP local: prérequis mobile

Les URLs locales actuelles sont en `http`, pas en `https`.

#### iOS

Ajouter une exception ATS dans `ios/Runner/Info.plist` pour le dev local :

```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
</dict>
```

#### Android

Autoriser le cleartext en debug/profile :

- `android/app/src/debug/AndroidManifest.xml`
- `android/app/src/profile/AndroidManifest.xml`

Contenu minimal :

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET"/>
    <application android:usesCleartextTraffic="true" />
</manifest>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/diliosinteractive) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
