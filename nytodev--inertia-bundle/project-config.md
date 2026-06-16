---
trigger: always_on
description: > **Ce fichier est la source de vérité pour tous les agents IA travaillant sur ce projet.**
---

# AGENTS.md — symfony-inertia-bundle

> **Ce fichier est la source de vérité pour tous les agents IA travaillant sur ce projet.**
> Il définit les conventions, contraintes et outils à respecter **sans exception**.

---

## Contexte du projet

Ce projet est un **Symfony Bundle** qui implémente le protocole Inertia.js v2 (avec chemin de migration v3)
côté serveur, l'équivalent de `inertiajs/inertia-laravel` mais pour Symfony.

- **Type** : Symfony Bundle réutilisable (pas une application)
- **Inertia.js** : v2 en priorité, architecture extensible vers v3
- **Symfony** : 6.4 LTS minimum → 7.4 LTS → 8.0
- **PHP** : ^8.1
- **Pattern** : `AbstractBundle` (Symfony 6.1+), **jamais** `Bundle` + `Extension` séparée
- **Packagist** : `nytodev/inertia-bundle` (type `symfony-bundle`)

---

## Structure du projet

```
symfony-inertia-bundle/
├── AGENTS.md               ← ce fichier
├── composer.json
├── LICENSE
├── README.md
├── config/
│   ├── definition.php      ← schéma de configuration (DefinitionConfigurator)
│   └── services.yaml       ← définitions de services explicites (pas d'autowire)
├── docs/
│   └── index.md
├── src/
│   ├── InertiaBundle.php   ← AbstractBundle : configure() + loadExtension()
│   ├── Service/
│   │   └── Inertia.php     ← service principal : render(), share(), defer(), once()
│   ├── Response/
│   │   └── InertiaResponse.php
│   ├── EventListener/
│   │   └── InertiaListener.php  ← kernel.request + kernel.response
│   ├── Props/
│   │   ├── LazyProp.php
│   │   ├── DeferProp.php
│   │   ├── OnceProp.php
│   │   └── MergeProp.php
│   ├── Twig/
│   │   └── InertiaTwigExtension.php
│   └── Controller/
│       └── AbstractInertiaController.php
└── tests/
    ├── Functional/
    └── Unit/
```

---

## Conventions PHP — OBLIGATOIRES

### Style de code
- **PHP 8.1+ uniquement** — utiliser les features modernes : `readonly`, `enum`, `fibers` si pertinent
- **Strict typing** : `declare(strict_types=1)` dans **chaque** fichier PHP
- **Types complets** sur toutes les méthodes (paramètres + retour), jamais `mixed` sans raison
- **Constructor promotion** pour la DI : `public function __construct(private readonly Foo $foo)`
- **`readonly`** sur toutes les propriétés qui ne mutent pas
- **`final`** sur toutes les classes sauf si l'extension est intentionnellement supportée
- **Comparaisons strictes** : toujours `===` et `!==`, jamais `==` ou `!=`
- **Yoda conditions** : `if (null === $value)`, pas `if ($value === null)`
- **`match`** plutôt que `switch` pour les expressions
- Nommage **camelCase** pour variables et méthodes, **PascalCase** pour classes

### Organisation des fichiers
- 200–400 lignes par fichier, **800 max**
- Une classe par fichier, toujours
- Pas de logique dans les constructeurs (sauf assignation de propriétés)
- Les exceptions dans un sous-namespace `Exception/`

### Interdit
- `var_dump()`, `print_r()`, `dd()` dans le code de production
- `@` pour supprimer les erreurs PHP
- Closures récursives non documentées
- Magic strings non-constantes répétées (utiliser des constantes de classe)

---

## Conventions Symfony — SPÉCIFIQUES AU BUNDLE

### DI et services
- **`AbstractBundle`** obligatoire — jamais `Bundle` + `Extension` séparée dans `DependencyInjection/`
- `configure()` → schéma importé depuis `config/definition.php`
- `loadExtension()` → charge `config/services.yaml` + injecte la config dans les services
- Services **préfixés** `inertia.*` (alias DI du bundle = `inertia`)
- Tous les services **`public: false`** par défaut
- Alias publics pour l'autowiring : `Nytodev\InertiaBundle\Service\Inertia: { alias: inertia.service, public: true }`
- **Zéro autowiring, zéro autoconfigure** dans les définitions de services du bundle
- Chemins **physiques** `__DIR__` dans `loadExtension()`, jamais de chemins logiques `@Bundle`

### Event Listeners
- Nommage : suffixe `Listener` (ex: `InertiaListener`), jamais `Subscriber` dans les classes
- Tag explicite : `{ name: kernel.event_subscriber }` dans services.yaml
- Listener unique pour `kernel.request` + `kernel.response`

### Configuration du bundle (config/packages/inertia.yaml)
```yaml
inertia:
    root_view: 'base.html.twig'   # template Twig racine
    version: null                  # version des assets (null = désactivé)
    ssr_enabled: false             # SSR via serveur Node.js
    ssr_url: 'http://127.0.0.1:13714'
```

---

## Protocole Inertia.js v2 — Ce que le bundle doit implémenter

### Page Object v2 (structure exacte)
```json
{
    "component": "User/Edit",
    "props": { "errors": {}, "user": {} },
    "url": "/user/123",
    "version": "abc123",
    "clearHistory": false,
    "encryptHistory": false,
    "deferredProps": {},
    "mergeProps": [],
    "prependProps": [],
    "deepMergeProps": [],
    "matchPropsOn": [],
    "onceProps": {},
    "scrollProps": {}
}
```

### Comportements critiques du protocole
1. **Première requête** → HTML avec `<div id="app" data-page='...'></div>`
2. **Requête XHR** (header `X-Inertia: true`) → JSON page object + header `X-Inertia: true` + `Vary: X-Inertia`
3. **Version mismatch** → `409 Conflict` + header `X-Inertia-Location: <url>` (GET uniquement)
4. **302 → 303** : convertir automatiquement après PUT/PATCH/DELETE

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nytodev/inertia-bundle](https://github.com/nytodev/inertia-bundle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
