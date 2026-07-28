---
trigger: always_on
description: There are things which aren't part of the language specification but rather common conventions used in tools built on top of the language.
---

# Conventions

There are things which aren't part of the language specification but rather common conventions used in tools built on top of the language.

## File Types

The Cooklang ecosystem uses the following file types:

- **`.cook` files** — recipes written as plain-text instructions with Cooklang markup.
- **`.menu` files** — meal plans that reference recipes. A `.menu` file is a valid Cooklang file that uses sections for days and recipe references to compose a plan.
- **`.shopping-list`** — a shopping list definition combining recipe references and free-hand ingredients. Hidden file (one per directory).
- **`.shopping-checked`** — an append-only log tracking which ingredients have been checked off. Hidden companion to `.shopping-list`.

### Menu Files

Menu files use sections to organise days (or meals) and recipe references to pull in dishes. Recipe paths are relative to the root recipe directory, without the `.cook` extension.

```cooklang
= Monday

@./mains/pasta carbonara{2}
@./sides/green salad{}

= Tuesday

@./mains/chicken stir fry{4}
@./sides/steamed rice{4}

= Wednesday

@./soups/minestrone{6}
@./breads/focaccia{1}
```

Sections can include dates in `YYYY-MM-DD` format. Applications recognise these dates and can surface a shortcut link to the relevant day's plan (e.g. highlighting today's menu).

```cooklang
== Day 1 (2026-03-07) ==

@./breakfast/shakshuka{4%servings}

== Day 2 (2026-03-08) ==

@./mains/chicken stir fry{4%servings}
```

## Scaling Referenced Recipes

When you reference another recipe with `@./path/to/recipe{quantity}`, the quantity controls how the referenced recipe is scaled:

1. **No units** — scales the whole recipe by the given factor. `@./bread{2}` makes double the recipe.
2. **Servings** — reads the referenced recipe's `servings` metadata and calculates a scaling factor to match. `@./pasta carbonara{4%servings}` for a recipe written for 2 servings will double all quantities.
3. **Units** (experimental) — reads the referenced recipe's `yield` metadata and calculates a scaling factor based on that. Only matching units are supported. For example, if a sauce recipe has `yield: 500%ml`, then `@./sauces/hollandaise{150%ml}` scales it down to produce 150 ml.

## Adding Pictures
You can add images to your recipe by including a supported image file (`.png`,`.jpg`) matching the name of the recipe in the same directory.
```sh
Baked Potato.cook
Baked Potato.jpg
```
You can also add images for specific steps by including a step number before the file extension.
```sh
Chicken French.cook
Chicken French.0.jpg
Chicken French.3.jpg
```

Alternatively, you can set an image URL in the recipe metadata using the `image` key (see [Canonical metadata](#canonical-metadata)).

## Canonical metadata

To use your recipes across different apps, follow the conventions on how to name metadata in common cases:

| Key | Purpose | Example value |
| --- | --- | --- |
| `source`, `source.name` | Where the recipe came from. Usually a URL, can also be text (eg. a book title). | `https://example.org/recipe`, `The Palomar Cookbook <urn:isbn:9781784720995>`, `mums` |
| `author`, `source.author` | The author of the recipe. | `John Doe` |
| `source.url`|The URL of the recipe if nested format is used.|`https://example.org/recipe`|
| `servings`, `serves`, `yield` | Indicates how many people the recipe is for. Used for scaling quantities. Leading number is used for scaling, anything else is ignored but shown as units. | `2`,`15 cups worth` |
| `course`, `category` | Meal category or course. | `dinner` |
| `locale` | The locale of the recipe. Used for spelling/grammar during edits, and for pluralisation of amounts. Uses ISO 639 language code, then optionally an underscore and the ISO 3166 alpha2 "country code" for dialect variants | `es_VE`, `en_GB`, `fr`  |
| `time required`, `time` or `duration` | The preparation + cook time of the recipe. Various formats can be parsed, if in doubt use `HhMm` format to avoid plurals and locales. | `45 minutes`, `1 hour 30 minutes`,`1h30m` |
| `prep time`, `time.prep`|Time for preparation steps only.|`2 hour 30 min`|
| `cook time`, `time.cook`|Time for actual cooking steps.|`10 minutes`|
| `difficulty`|Recipe difficulty level.|`easy`|
| `cuisine`|The cuisine of the recipe.|`French`|
| `diet`|Indicates a dietary restriction or guideline for which this recipe or menu item is suitable, e.g. diabetic, halal etc.|`gluten-free`, or array of values|
| `tags`|List of descriptive tags.|`[2022, baking, summer]`|
| `image`, `images`, `picture`, `pictures`|URL to a recipe image.|`https://example.org/recipe_image.jpg` or array of URLs|
| `title`|Title of the recipe.|`Uzbek Manti`|
| `introduction`, `description`|Additional notes about the recipe.|`This recipe is a traditional Uzbek dish that is made with a variety of vegetables and meat.`|

## Shopping Lists

### Shopping list format

A shopping list is a pair of hidden files in a directory:

- **`.shopping-list`** — the list definition (recipe references and free-hand ingredients)
- **`.shopping-checked`** — an append-only log of checked/unchecked ingredients


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cooklang/spec](https://github.com/cooklang/spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
