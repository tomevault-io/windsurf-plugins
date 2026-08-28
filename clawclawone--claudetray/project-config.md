---
trigger: always_on
description: App macOS en barre de menu, SwiftUI natif, macOS 14+, **zéro dépendance externe**.
---

# ClaudeTray — notes de travail

App macOS en barre de menu, SwiftUI natif, macOS 14+, **zéro dépendance externe**.
Elle affiche l'usage du quota Claude (abonnement Max) : fenêtre 5 h, fenêtre hebdomadaire,
et une colonne par modèle limité (FABLE, OPUS… selon ce que renvoie l'API).

Version courante : **1.6**. Dépôt public sous licence MIT, releases sur GitHub.

Tout ce dépôt est public, ce fichier compris : il tient lieu de guide de contribution. Aucun secret,
aucun identifiant Apple, aucun chemin local n'y entre — les seuls identifiants du dépôt sont les
gabarits `"ton@email"` / `"XXXXXXXXXX"` du script de build.

Le cahier des charges d'origine est dans `ClaudeTray.md`. Le `README.md` — rédigé en anglais, il
s'adresse aux utilisateurs — couvre l'installation, la signature et les décisions de conception.
`CHANGELOG.md` sert de notes de release. Ce fichier ne répète aucun des trois : il liste ce qui
casse si on l'ignore.

## Prérequis utilisateur

Claude Code installé **et connecté** (`claude` puis `/login`) : c'est la connexion, pas
l'installation, qui écrit l'entrée de trousseau `Claude Code-credentials`. L'app Claude de bureau
ne convient pas — elle ne dépose qu'une clé Electron `Claude Safe Storage`, inexploitable ici.

## Règles à ne pas enfreindre

- **Deux destinations sortantes, pas une de plus** : `https://api.anthropic.com/api/oauth/usage`
  pour les quotas, et `https://api.github.com/repos/ClawClawOne/ClaudeTray/releases/latest` pour la
  vérification quotidienne de version, anonyme et débrayable (`updateCheckEnabled`). Aucune
  télémétrie, aucun paquet tiers, aucun SDK. Toute autre dépendance réseau est un bug.
- **Header `anthropic-beta: oauth-2025-04-20` obligatoire.** Sans lui, l'endpoint répond 401.
- **Jamais de `ANTHROPIC_API_KEY`.** C'est un abonnement Max, pas une clé à la consommation.
- **Le token n'est jamais mis en cache en mémoire.** Celui du trousseau expire en ~1 h et Claude
  Code le réécrit ; `TokenResolver.resolve()` est rappelé à chaque appel réseau.
- **Notifications déclenchées au franchissement, jamais sur un état.** Une notification part
  uniquement si le pourcentage était sous le seuil au relevé précédent et l'atteint au relevé
  courant. Ne jamais ré-armer sur `resets_at` : la date de reset de la fenêtre 5 h avance à chaque
  appel, ce qui renvoyait une notification à chaque rafraîchissement (bug 1.1).
- **La source du token est publiée à la résolution, pas au succès.** `UsageStore.fetchOnce` fixe
  `tokenSource` avant la requête. Sinon un token manuel refusé laisse à l'écran la source du dernier
  succès (le trousseau), et l'app a l'air d'ignorer le token collé alors qu'elle vient de l'utiliser
  et de se faire refuser (bug 1.2).
- **Le token manuel prime tant que le fichier existe**, même s'il déclenche un 401. Le retour au
  trousseau passe par la suppression du fichier : « Effacer le token manuel », qui ne touche à rien
  d'autre, ou « Révoquer l'accès au token », qui supprime le fichier *et* retire ClaudeTray des
  applications de confiance de l'item de trousseau (depuis la 1.4).
- **Ne jamais toucher aux entrées de trousseau des autres.** L'item `Claude Code-credentials`
  appartient à Claude Code. `KeychainAccessRevoker` ne retire que les applications de confiance dont
  le chemin désigne ClaudeTray, et n'écrit rien s'il n'y en a aucune. La liste de partitions
  (`teamid:`, `apple-tool:`) est laissée intacte : la réécrire risquerait de couper Claude Code de
  ses propres identifiants.
- **Le libellé de la barre de menu garde une géométrie et une identité constantes.** `MenuBarExtra`
  re-présente sa fenêtre quand la vue du libellé change de forme : un pictogramme qui apparaît, une
  branche `if/else` qui remplace une `Image` par un `Text`, et le popover se rouvre tout seul après
  avoir été ouvert une fois dans la session. Le défaut existait depuis la 1.0 et n'est devenu visible
  qu'en 1.6, quand le pictogramme d'erreur a rendu le changement de largeur fréquent ; corrigé dans
  la même version. L'emplacement d'état est donc toujours présent, transparent quand il n'y a rien à
  signaler, et le rendu passe toujours par `Image`.
- **Un échec de vérification de version n'est pas « à jour ».** `UpdateChecker.check()` renvoie
  trois issues distinctes ; les confondre dans un même `nil` faisait annoncer « ClaudeTray est à
  jour » alors que la requête n'avait rien atteint (bug 1.5).
- **Aucun reset calculé localement.** On affiche `resets_at` tel quel, ou « non communiqué ».
  Le comportement réel de la fenêtre hebdo est instable : une prédiction fausse est pire que rien.
- **Une erreur n'efface jamais les données.** Le dernier instantané valide reste à l'écran, avec
  un message lisible et un marqueur « obsolète depuis X » au-delà de 15 min.
- **Respecter la cadence.** L'endpoint renvoie des 429 persistants s'il est trop sollicité :
  cadence minimale 5 min, backoff exponentiel plafonné à 30 min, `Retry-After` prioritaire.
  Le mode « Auto » (90 s quand la fenêtre 5 h était entamée) a été retiré en 1.6 : il provoquait
  des 429 en série. Ne jamais réintroduire d'option sous les 5 minutes.
- **Toute chaîne visible passe par `Loc`.** Aucune chaîne d'interface écrite en dur dans une vue :

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClawClawOne/ClaudeTray](https://github.com/ClawClawOne/ClaudeTray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
