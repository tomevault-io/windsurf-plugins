---
trigger: always_on
description: <!-- start: Packmind standards -->
---

<!-- start: Packmind standards -->
# Packmind Standards

Before starting your work, make sure to review the coding standards relevant to your current task.

Always consult the sections that apply to the technology, framework, or type of contribution you are working on.

All rules and guidelines defined in these standards are mandatory and must be followed consistently.

Failure to follow these standards may lead to inconsistencies, errors, or rework. Treat them as the source of truth for how code should be written, structured, and maintained.

# Standard: SolidAI — Aggregate Domain : Racine, Entités et Value Objects

Standardize each SolidAI domain module as a single-aggregate structure with one dataclass aggregate root at the module root (Trio Atomique: intention.md + implementation + _tests/), entities in entities/, immutable identityless value objects in value_objects/, and immutable domain events in events/ (each with its own Trio Atomique) to make aggregate boundaries, invariants, and responsibilities immediately navigable and unambiguous. :
* Implémenter l'aggregate root comme dataclass portant les invariants métier et exposant les méthodes de mutation de l'agrégat — sans logique d'infrastructure ni import hors domain/
* Isoler chaque entité de l'agrégat dans domain/X/entities/ avec son propre Trio Atomique — une entité a une identité propre mais n'est pas la racine de l'agrégat
* Isoler chaque value object dans domain/X/value_objects/ avec son propre Trio Atomique — un value object est immuable, sans identité, défini uniquement par sa valeur
* Isoler les événements domain de l'agrégat dans domain/X/events/Y/ avec leur propre Trio Atomique — un événement domain est immuable, représente un fait passé, et ne dépend que du domain/
* Placer exactement un seul fichier de code à la racine du module domain — l'aggregate root — accompagné de son intention.md et de son dossier _tests/ ; ne jamais y placer d'entités ni de value objects

Full standard is available here for further request: [SolidAI — Aggregate Domain : Racine, Entités et Value Objects](.packmind/standards/solidai-aggregate-domain-racine-entites-et-value-objects.md)

# Standard: SolidAI — Aggregate Domain : Racine, Entités, Value Objects et Events

Standardiser l’anatomie des modules domain SolidAI en structurant chaque agrégat avec un unique aggregate root Python dataclass à la racine (Trio Atomique <module>_intention.md + implémentation + _tests/), des entités/value objects/events isolés dans entities/, value_objects/ et events/ avec leur propre Trio Atomique, et des interfaces dans repositories/ sans imports hors domain/ ni logique d’infrastructure afin de préserver les invariants, éviter les collisions de noms et améliorer la maintenabilité. :
* Créer un sous-dossier repositories/ à la racine du module domain pour y placer les interfaces de repository de l'agrégat — ces interfaces expriment le contrat de persistance du domain sans dépendance infrastructure
* Implémenter l'aggregate root comme dataclass Python portant les invariants métier et les méthodes de mutation — sans import hors domain/ et sans logique d'infrastructure
* Isoler chaque entité de l'agrégat dans domain/X/entities/Y/ avec son propre Trio Atomique (<entity>_intention.md, <entity>.py, _tests/) — une entité a une identité propre mais n'est pas la racine de l'agrégat
* Isoler chaque événement domain de l'agrégat dans domain/X/events/Y/ avec son propre Trio Atomique (<event>_intention.md, <event>.py, _tests/) — un événement est immuable et représente un fait passé dans le domain
* Isoler chaque value object dans domain/X/value_objects/Y/ avec son propre Trio Atomique (<vo>_intention.md, <vo>.py, _tests/) — un value object est immuable, sans identité, défini uniquement par sa valeur
* Placer exactement un seul fichier de code à la racine du module domain — l'aggregate root — avec son <module>_intention.md et son dossier _tests/ ; ne jamais y placer d'entités, value objects ou events

Full standard is available here for further request: [SolidAI — Aggregate Domain : Racine, Entités, Value Objects et Events](.packmind/standards/solidai-aggregate-domain-racine-entites-value-objects-et-events.md)

# Standard: SolidAI — Architecture Fractale Trio Atomique : Intention, Code, Tests Co-localisés

Enforce the SolidAI “Trio Atomique” fractal module layout—<module>_intention.md (Rationale/Responsibility/Design), <module>.py, and co-localized _tests/<module>_test.py at distance 1 (never .test.py)—to standardize DDD architecture, preserve Python import resolution, and make intent, implementation, and validation auditable while minimizing architectural debt. :
* Appliquer la structure du trio atomique (<module>_intention.md + <module>.py + _tests/) de manière uniforme à toutes les couches DDD sans exception
* Créer un fichier intention.md pour chaque module en utilisant le template Rationale / Responsibility / Design, même si le contenu est vide au départ
* Nommer les fichiers de test avec le suffixe _test.py (underscore) et jamais .test.py (point) — le point dans un nom de fichier Python casse la résolution des imports
* Placer les tests dans un sous-dossier _tests/ à l'intérieur du dossier du module — c'est le principe de distance minimale : chaque fichier est à exactement 1 niveau de son test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Luciosmic/AEFI_Acquisition](https://github.com/Luciosmic/AEFI_Acquisition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
