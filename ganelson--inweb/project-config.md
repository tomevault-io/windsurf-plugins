---
trigger: always_on
description: A _convention_ is a sort of preference setting which changes the normal way
---

# Conventions

A _convention_ is a sort of preference setting which changes the normal way
a web is woven, tangled, or simply read in. There are relatively few of these,
and the generic values are usually sensible, so most webs will never need
to change them.

The currently applicable conventions can be seen using `inweb inspect -conventions`:

``` ConsoleText
	$ inweb inspect -conventions smorgasbord
	web "Sorting Smorgasbord" (Python program in MarkdownCode notation): 2 sections : 10 paragraphs : 114 lines
	(generic)  paragraph numbers are visible
	(generic)  namespaces are unenforced
	(generic)  sections are not numbered sequentially
```

...and so forth. For example, `paragraph numbers are visible` is a convention.
`(generic)` means that it applies because this generically applies to every web,
unless Inweb is given any instructions to the contrary.

Conventions can be specified on the contents page of a web. For example, we could
rewrite `Contents.inweb` for `smorgasbord` to read:

	Title: Sorting Smorgasbord
	Author: Various Artists
	Notation: MarkdownCode
	Language: Python
	
	Sections
		Counting Sort
		Quick Sort

	Conventions {
		sections are numbered sequentially
	}

Note that conventions appear after the contents rundown, and in braces. (That's
because this is technically a _declaration_: see //Resources and Declarations//.)
We then see:

``` ConsoleText
	$ inweb inspect -conventions smorgasbord
	web "Sorting Smorgasbord" (Python program in MarkdownCode notation): 2 sections : 10 paragraphs : 114 lines
	(generic)  paragraph numbers are visible
	(generic)  namespaces are unenforced
	(web)      sections are numbered sequentially
```

...and so on. Note that `(web)`, showing that the web makes the convention this
time. (For the effect this change actually has, see below.)

Conventions can be made, in fact, in five ways:

- `(generic)` means this is Inweb's default;
- `(language)` means that the language, in the above example `Python`, chose this;
- `(notation)` means that the web's notation, in the above example `MarkdownCode`, chose this;
- `(-using)` means that a file of conventions read by `-using FILE` chose this;
- `(colony)` means that the colony, if there is one, made the choice;
- and `(web)` means that the web's contents page said so.

In this list, later sources beat earlier ones. (`inweb inspect -conventions -fuller`
shows exactly how this trail is resolved.)

So, for example, if you have many unrelated programs which you want to set some
common preferences for, or if you want to set preferences for a single-file web
which has no contents page, the best way is probably to create a standalone preferences
file — called, say, `prefs.inweb`. This could for example read:

	Conventions {
		namespaces are enforced
		holon names are written between ▶️ and ◀️
	}

Whereupon `inweb weave program.c.md -using prefs.inweb` would have these
conventions applied, but `inweb weave program.c.md` would not.

On the other hand, if you want to apply some conventions across a set of programs
which are closely related, it's probably better to create a colony for them
(see //Colonies//) and then include conventions at the foot of the colony file.

## Conventions changing how webs are read in

* `commentary uses Markdown markup`, `commentary uses simplified markup`,
  or `commentary uses TeX markup`. Here, something confusing must be explained:
  Inweb supports multiple notations for webs, including the one called `MarkdownCode`
  which is used in most of this guide. Other notations look very different.
  But all of them feature some sort of markup features in their
  commentary: some way to indicate that text should be italicised, for example.
  So it can be the case that `commentary uses Markdown markup`, even
  if the notation is not `MarkdownCode`. Older Inweb webs have traditionally
  used `commentary uses simplified markup`, which as the name suggests, is
  more restricted. At present, `commentary uses TeX markup` is experimental
  and not fully working. Generically, `commentary uses simplified markup`,
  but both of the main notations supplied with Inweb now specify that
  `commentary uses Markdown markup`, so in practice that's what users will
  see unless they make a choice otherwise.

* `a summary under the title is read as the purpose`,
  `an italicised summary under the title is read as the purpose`, or
  `a summary under the title is not read as the purpose`.
  See //Metadata//, which describes what `an italicised summary under the title is read as the purpose`
  does, since that's the normal setting for Markdown-notation webs. Older
  Inweb webs tended to use `a summary under the title is read as the purpose`,
  which doesn't require the summary to be in italics, but does require it to
  be the only text "in limbo", that is, occurring before the first explicit
  paragraph opening. (This makes no sense for Markdown because there are no
  explicit paragraph markers in that notation.) Generically, though,
  `a summary under the title is not read as the purpose`; notations must
  actively decide to allow this.

* `holon names can be abbreviated`, `holon names can be abbreviated even at declarations`,
  or `holon names cannot be abbreviated`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ganelson/inweb](https://github.com/ganelson/inweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
