---
trigger: always_on
description: We have all of our source assets separated in file directory *as well as* repositories. Since art/content folks will be modifying source-side content (such as TGAs/PSDs/FBXs), we don't want to require them to download the entire game project as well.
---

# Unreal Engine 4 - Joy Machine Style Guide

## Overview
We have all of our source assets separated in file directory *as well as* repositories. Since art/content folks will be modifying source-side content (such as TGAs/PSDs/FBXs), we don't want to require them to download the entire game project as well.

For *Steel Hunters*, for instance, we have: 
* `steelhunters` - The entire UE project folder, with file assets consisting solely of UASSETS/UMAPs.
* `steelhunters-content` - All of our source assets that we use to import into UE as UASSETS.
* `joyengine` - Our fork of Unreal Engine 4, which has an extensive amount of changes and additions from vanilla UE4. As such, you cannot use any UE4 binaries downloaded from the Epic Game Launcher.

### Version Control
* Most day-to-day, non-feature-sized work is done in `develop`.
* For major features, work within a feature branch (`feature/my-branch-name`). There are also branch specifiers for:
	- Bug-Fix (non-trivial fix to a non-release build): `bugfix/`.
	- Hot-fix (non-trivial fix to a release build): `hotfix/`.
	- Release (I honestly don't know why this exists): `release/`.
	- Support (Also: no idea): `support/`.
* We will generally make tags of each repository's state whenever it's in a generally stable state. 
	- This is Trent's responsibility.
* We will push to master when a tagged repository is confirmed to be in a stable state.
	- Also Trent's responsibility.
 
* **IMPORTANT NOTE**: If at any point you're in a weird local state, don't feel comfortable merging something into `develop`, or any other questions: ask. Ask. ASK. ask ask ask. 

### Code-Side Conventions.

#### Game Code Conventions.
* Follow standard UE4 practices: 
	- (Most) headers go into the `Public` directory.
	- All source (C/CPP) files go into the `Private` directory.
* Any third-party code-only libraries go into a `ThirdParty` folder (in `Public` and `Private`).
* All C/C++ files must first include `steelhunters.h`. You'll know if they don't because the project won't build.
* With the rare (or particularly small) exception, stick to one class declaration/definition per file set. 
* I can't recall if this is enforced by the UE4 build tool, but if you're making an actor or component (`AWhatever`/`UWhateverComponent`), the initial letter designation does not get used in the filename. Instead, append the written-out form to the end of the filename (`WhateverActor`/`WhateverComponent`). 
	- Actors do not require `Actor` in the name of the class (they are prefixed with `A`).
	- Interfaces do not require `Interface` in the name of the class (they are prefixed with `I`).
	- Components _do_ require `Component` in the name of the class (they are prefixed with `U` as well).
	- General structures/classes that don't fit into the aforementioned types are prefixed with `F`.
	- Enumerations are prefixed with `E`.
		* The contents of an enumeration are, if possible, prefixed with the acronym for the name of the enumeration (minus the `E` prefix).
  		* All enumerations end with a `_MAX` entry.
* All class members and function arguments are camel case.
 	- If the argument can potentially be a conflict with a class member, append `In` to it.
 	- If an argument is intended to be filled by a method to use outside of it, append `Out` to it.
* All UE-related (i.e. not _purely_ internal code) classes/structures/enumerations/methods/properties are preceded by `UCLASS`/`USTRUCT`/`UENUM`/`UFUNCTION`/`UPROPERTY`.
 	- Additionally, all classes (and most structures) first in-declaration line is `GENERATED_BODY`.
 	- Apply the appropriate properties and metadata designations for each of these.
* Function/method local variables are lower camel case.
* Use `uint32` with a bitmask specifier (i.e. `uint32 MyFlag : 1`) in most cases. 
 	- Exception to this rule is is a boolean value is required for an `EditCondition` property, in which case use `bool`. 
 	- For the love of all that is holy, do not prefix boolean values of any kind with `b`. 
* For `UPROPERTY` members with the `Transient` property, put them in their appropriate category, but under a nested `Runtime`:
 	- For members that are in the `Core` group, a transient member would be in `Category = "Core|Runtime"`.
* Variables rarely should require a comment to explain what they are.
	- This doesn't mean you have to be verbose for the sake of explicitness, of course, just organize a class/function and then read the contained data and see if everything makes sense given the context.
* _Don't_ add comments for the sake of adding comments.
	- If a variable is named `IgnoreIncomingDamage`, don't comment it with `// When enabled, ignore all incoming damage.`.
* _Do_ add comments to demarcate sections of code, especially in class/structure declarations. 
* Typos in variable names happen (but fix them); consistent misspellings do not.
* Use constants for any known strings. Do not just type them in the middle of a function.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trentpolack/CALVINBALL](https://github.com/trentpolack/CALVINBALL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
