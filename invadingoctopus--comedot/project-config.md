---
trigger: always_on
description: 1. Underscores == ugly
---

# Comedot Coding Conventions & Design Guidelines

## Axioms of Goom

1. Underscores == ugly

2. Tabs > Spaces

3. camelCase == bestCase

This is the Truth of the Universe.

----


> [!TIP]
> Most of this is the opinion of the sole maintainer @ShinryakuTako on GitHub but if you're smart it should be yours too.

I come from Swift and I love it so this framework attempts to mimic the Swift API Design Guidelines unless when it's highly inconvenient within Godot: https://www.swift.org/documentation/api-design-guidelines/


## Voidspace

* Tabs instead of Spaces
	- Because GDScript is an indentation-based language :)
	- A single missing or extra space could cause errors.
	- Visual representation can be customized per user and easier to view with visible tabs etc.
	- Easier to navigate.
	- Fewer bytes to store.

* 2 empty lines between functions and different regions of code, such as parameters, signals, state properties etc.
	- This is what the default Godot script templates start with.
	- Adds more clear visual separation between distinct sections.


## Case

* NO underscores whenever possible!
	- Less clutter
	- Fewer keystrokes  
	(Thanks Stannis!)

* camelCase for everything, even constants.
	- No extra SHIFT press needed to start autocomplete etc.

* Capitalized names for Types only.

* Short acronyms may be fully capitalized.
	- Examples: UINode, HUDColor

* Text names/IDs such as for node groups, input actions and animations should be camelCase, to match the convention of enums: `GlobalInput.Actions.yeet = &"yeet"`


## Names

* Names should make grammatical sense wherever possible.

* Booleans should start with `is`, `has`, `should` etc.
	- This may make autocompletion easier by listing all booleans together.
	- Avoid ambiguity with "verbs" e.g.`showDebugInfo` could be a function name.
	- `bool` parameters in function/methods calls may be named as verbs/commands to match the function convention e.g. `skipEmptyCells` in `randomizeTileMapCells()`

* Components that manage a collection of a certain Resource may pluralize the name of that resource, such as `StatsComponent` because `StatComponent` may be ambiguous and imply that it is a certain specific single stat (such as `HealthComponent` which specifically names the state).


### Functions & Methods

* Function names should read like a verb/command/action: e.g. `doSomething()` or `checkValidity()`

* Functions that perform a quick & "cheap" retrieval operation, like returning a member from an Array or Dictionary, should be named starting with `get`: e.g. `getComponent(…)`

* Functions that need to do a slower _search_ operation, like scanning a list of all child nodes, should be named starting with `find`: e.g. `findComponent(…)`

* Functions that add an _existing_ object to a parent, container or list, should be named starting with `add`: e.g. `addText(…)`

* Functions that _create_ a new object and then add it to a parent, should be named starting with `create`: e.g. `createLabel(…)`

* Godot is a dummy about not having argument labels, so some arguments like `true`/`false` may be mysterious and ambiguous at the call point; add labels manually in a trailing comment: e.g. `doSomeAction(someObject, true, false) # logResult, skipCooldown`


### Signals

* Signals should generally be named in this form: {object/category}{tense}{event} e.g. `healthDidDecrease`
* or, if the ACTION is the focus: {action}{object} e.g. `didSpawnEntity`
* or, if the OBJECT is the focus: {object}{action} e.g. `entityDidSpawn`
* Signal names should begin with a `did` or `will` wherever it makes sense.
	- This allows the event handlers to know exactly at what point the effects of their code will occur.
	- This ensure consistency in words by reducing English jankery: `didDecrease` vs `decreased`, `didRunOut` vs `ranOut`
	- `ammoInsufficient` does not make sense in a past or future tense, so it is exempt.
	- If there are no "did" or "will" variants the tense can be omitted, e.g. `onCollide`.

_Examples:_
```
signal healthDidZero
signal didFire(bullet: Entity)
signal didSpawn(newSpawn: Node2D, parent: Node2D)
signal willRemoveFromEntity
```

* Functions that handle signals should be named in this form: `on[ObjectThatEmittedSignal]_[signal]`
	- If the script is attached to the node which emits the signal, then simply: `on[Signal]`
	- If the object name is a short single word, then the _ underscore may be omitted: `onAreaEntered` instead of `onArea_entered`
	- Yes, this is the ONLY place where underscores are tolerated, because we can't use a — dash etc. :')

_Examples:_
```
func onCollectibleComponent_didCollideCollector(…)
func onGunComponent_didDepleteAmmo()
func onHealthChanged(…)
func onTimeout() # in the script of a Timer node
```

### Files

* Filenames should be clear and precise. 
* Add suffixes like `Entity` and `Component` to assist referencing and searching etc.: `MonsterEntity.gd`, `MonsterAttackComponent.gd` etc.
	- There may be exceptions for brevity for certain resources such as `Health.gd` instead of `HealthStat.gd` unless there is ambiguity.
* Filenames should be _concise_ but they don't have to be _short:_ e.g. `TurnBasedTileBasedPlatformerControlComponent` :')

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InvadingOctopus/comedot](https://github.com/InvadingOctopus/comedot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
