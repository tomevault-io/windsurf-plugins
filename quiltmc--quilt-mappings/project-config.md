---
trigger: always_on
description: Use `UpperCamelCase` for class names. Use `lowerCamelCase` for method names, variable names, and names of fields that are not
---

# Naming conventions

## General

Use `UpperCamelCase` for class names. Use `lowerCamelCase` for method names, variable names, and names of fields that are not
both static and final. Use `UPPER_SNAKE_CASE` for names of fields that are both static and final, excluding atomics like
`AtomicInteger`, `AtomicBoolean` or `AtomicReference`.

Method names should generally be verb phrases (`tick`, `getCarversForStep`), except for "withX", "toX", "fromX", "of" and
builder methods. Class names and non-boolean field and variable names should be noun phrases (`ChunkRegion`, `color`).
Boolean field and variable names should always be adjective phrases or present tense verb phrases (`powered`, `canOpen`),
avoiding the `is` and `has` prefixes when possible (`colored`, not `isColored` or `hasColor`).

To make code as easy to read as possible, keep names in the natural language order. For example, a class representing a chest
block entity should be named `ChestBlockEntity` rather than `BlockEntityChest`. Though prefix naming may be helpful for
grouping classes together in an IDE's tree view, reading and writing code is done much more often than browsing files.

## Spelling

Use American English for consistency throughout Quilt Mappings and with known Mojang names.

If there are two acceptable spellings of the same word, first check if one is already being used in Quilt Mappings or by Mojang, and if
not, use the most common spelling.

## Conciseness

Omit words that are made redundant by parameter names or owner class names. For example, use `getChunk(BlockPos pos)` rather
than `getChunkAtPosition(BlockPos pos)` and `Box.create` rather than `Box.createBox`. Don't avoid overloading methods or
shadowing fields.

However, it's more important for a name to be descriptive rather than short, so don't omit important words. When naming
something always look at all its usages, including overriding methods and inheriting classes.

It's important to be concise especially with names used in many places throughout the code, while more obscure names can be
longer and more descriptive.

## Abbreviations

Avoid abbreviations unless it's a common one everyone knows and other Quilt Mappings names involving the same word use its abbreviated
form. Full names are easier to read quickly and remember ("Which words were abbreviated?") and they often don't take more
time to type thanks to IDE autocompletion. Common abbreviations you should use are:

 - "id" for "identifier"
 - "pos" for "position"
 - "nbt" for "named binary tag"
 - "init" for "initialize"
 - "min"/"max" for "minimum"/"maximum"
 - Any abbreviations used by Java or libraries ("json", "html", etc.)
 - "o" for the parameter of `equals(Ljava/lang/Object;)Z` methods
 - "prev" for "previous"

Treat acronyms as single words rather than capitalizing every letter. This improves readability (compare `JsonObject` and
`JSONObject`) and it's consistent with Mojang naming (a known name is `NbtIo`).

Abbreviations you shouldn't use are:
- "loc" for "location"

## Packages

Package names should always be singular to respect Java conventions. Try to respect the Mojang package structure to avoid
visibility problems in the future. For multiple-word packages, attempt to set up a structure where the words can be separated
into two packages: for example, recipe book related classes are in `recipe.book`. If this isn't possible, use underscores to separate,
as used in `feature_flag`.

## Consistency

Consistency is important as it makes code more readable and names easier to memorize. When possible, use terms that are present
in other Quilt Mappings names, in libraries used by Minecraft, or in vanilla strings. The rest of this section lists common names and
name patterns you should use.

### Ticks and updates

Use "tick" for updates done once per tick. Use "update" for other kind of updates.

### Value last tick

Use the word "last" for the value that something had last tick (`lastX`, `lastWidth`, etc.).

### Getters, setters, withers, and creators

Use "get" for non-boolean getters and other methods that calculate some property with no side effects other than caching a value
in a private field. For boolean getters, use "is".

Use "set" for methods that set some property. Name the parameter the same as the property (`setColor(color)`, not
`setColor(newColor)`).

Use "with" for methods that return a copy of an object with a different value for some property. Name the parameter the same
as the property.

Use "create" for methods that create a new instance of some object. Use "get or create" for methods that create a new
instance only if one does not already exist. Don't use "get or create" for lazy initialization, though. 

### Serialization

Use "serializer" for objects whose purpose is serializing or deserializing some type of object (`RecipeSerializer`). Use
"serialize" and "deserialize" for methods only when serializing or deserializing an object other than the one the method is in.

Use "from" for static methods that create an object of the method owner's type (`fromJson`, `fromNbt`, `fromString`). Use "to"
for methods that convert an object to another type (`toString`, `toLong`, `toNbt`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuiltMC/quilt-mappings](https://github.com/QuiltMC/quilt-mappings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
