---
trigger: always_on
description: Pépin est un **CSPM multi-cloud souverain** (Go). Il évalue la **posture** d'un
---

# CLAUDE.md — règles de travail assisté par IA sur Pépin

Pépin est un **CSPM multi-cloud souverain** (Go). Il évalue la **posture** d'un
cloud (OVH, Scaleway, Exoscale, Outscale…) contre un **référentiel commun** de
contrôles (`referentiel/`), ancré sur **SCSL** (module posture cloud), **SecNumCloud**,
**CIS** et **ISO**. « Pépin trouve les pépins de votre cloud. »

> Né de la généralisation d'**osc-policy** (Outscale) et de la lignée **pitstop**
> (SCSL-R) / **plumber**. Le référentiel commun est la source de vérité des
> contrôles ; les règles Rego par provider en sont l'implémentation.

## 0. Règle d'architecture FONDAMENTALE — un seul jeu de règles commun

**Toutes les règles de posture sont COMMUNES à tous les providers.** Elles vivent
dans **`internal/commonrules/`** et s'évaluent sur le **modèle normalisé commun**
(`input.resources[]`, types et attributs agnostiques). **Aucune règle n'est propre
à un provider.** Ce qui change d'un cloud à l'autre, c'est uniquement la **SOURCE**
(le collecteur) : tout est **normalisé AVANT** de passer dans les règles.

- **Providers = COLLECTEURS Go compilés.** Un cloud = un package `providers/<nom>/`
  qui implémente `provider.Provider` (interface = `Name`/`Description`/`Collect`
  [+ `TerraformMapper`]). Son seul rôle : projeter sa source (API live via SDK
  natif, plan Terraform, S3) vers le **modèle normalisé commun**. Il ne porte
  **aucune** règle Rego. Ajout d'un provider ⇒ recompilation. Skill `nouveau-provider`.
- **Règles = Rego commun.** `internal/commonrules/rules/*.rego` (embarqué, chargé à
  chaque scan) + règles externes `--policy-dir` (hot-load), fusionnées et
  interrogées via `data.pepin.rules.deny`. `labels.provider` est **tiré de la
  ressource** (`provider_of(r)`), jamais codé en dur. Une règle ne se déclenche que
  si des ressources du type visé existent ; un provider qui n'a pas ce type ne la
  déclenche pas (pas de faux positif). Skill `nouvelle-regle`.

> Corollaire : pour couvrir un nouveau check, on **normalise** la donnée dans le
> collecteur/mapper vers le modèle commun, puis on écrit **une** règle commune —
> jamais une règle par provider.

## 1. Build, test, audit — toujours via `mise`

Le projet est piloté par **mise** (`mise.toml`, plus de Makefile) : `mise run <tâche>`
(`mise tasks ls` pour la liste ; mise fournit aussi les outils — opa, terraform).

```bash
mise run build   # compile (version injectée)
mise run test    # go test ./... -race + tests Rego (opa)
mise run audit   # vet + lint (golangci-lint) + sec (gosec) + vuln (govulncheck) + osv
```

Ne pas committer si `mise run test` ou `mise run audit` échoue.

### 1.1 Scans réels & provisionnement (comptes Outscale + Scaleway disponibles)

Des comptes **Outscale** et **Scaleway** réels sont disponibles pour valider une
règle par un scan live (`--live`), condition non négociable avant d'ACTIVER un
contrôle (`fournisseurs:` non vide, contrat `verifie`).

**RÈGLE ABSOLUE — nettoyage : toute ressource provisionnée pour un test DOIT être
détruite à la fin** (`terraform destroy`, ou suppression via l'API/console). Ne
jamais laisser une ressource de test vivre : coût, surface d'exposition, dérive.
Tenir la liste de ce qui est créé et confirmer la destruction avant de conclure.

**Préférer le NON-provisionnement.** pepin sait auditer un **plan Terraform**
(`scan --tf plan.json`, chemin `tfmap`/`tfparse`) : pour valider une règle et le
mapping d'un nouveau type, écrire (ou réutiliser depuis GitHub) un `.tf` du/des
provider(s) souverain(s), générer `terraform plan -out` → `plan.json`, et scanner
CE plan — aucune ressource cloud créée. Le scan live ne sert qu'à confirmer le
**contrat d'API** (champs réels) quand le plan ne suffit pas ; il est alors suivi
d'un `destroy` immédiat.

### 1.2 Bilinguisme : docs, CLI et référentiel

Comme pavois, les docs du dépôt sont **bilingues FR/EN**. **L'anglais est la langue
PRIMAIRE** : `README.md`, `SECURITY.md`, `CONTRIBUTING.md` sont en anglais ; leur
contrepartie française est `*.fr.md`, reliée par un sélecteur de langue en tête
(`🇬🇧 English · [🇫🇷 Français](*.fr.md)`). Tenir les deux versions synchronisées à
chaque changement.

**La CLI et le RÉFÉRENTIEL sont eux aussi bilingues** (issue #37) : la langue est
DÉTECTÉE (`--lang` → `PEPIN_LANG` → `LC_ALL` → `LANG` → repli `en`, cf.
`internal/i18n`), et elle vaut pour tout ce que l'outil imprime (rapport, verdict,
aide, erreurs) comme pour les formats parsables (`json`, `sarif`, `oscal`,
`assessment`). Le **français reste la langue de RÉFÉRENCE du contenu normatif** :
c'est lui qui fait foi, l'anglais en est la traduction maintenue en parallèle.

Concrètement, tout texte destiné à l'utilisateur s'écrit DEUX fois, côte à côte :

- **Go** : `i18n.T(fr, en)` au point de rendu ; les chaînes d'aide de cobra, figées
  à l'init, sont réécrites par `cmd.localize()` après résolution de la langue.
- **Rego** : `message`/`remediation` en français, `labels.message_en` et
  `labels.remediation_en` en anglais (`finding.Finding.Labels` est extensible :
  **on ne modifie pas scankit**). Le scan consomme ces labels puis les RETIRE, car
  ce sont un transport, pas une donnée du rapport.
- **Référentiel** : `titre_en`, `description_en`, `remediation_en` à côté de leurs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stephrobert/pepin](https://github.com/stephrobert/pepin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
