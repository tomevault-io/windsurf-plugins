---
trigger: always_on
description: Module organization and API design guidelines
---

---
source: https://www.haskellforall.com/2021/05/module-organization-guidelines-for.html
---

The following is for Haskell, but also largely applies to Elm:

Module organization guidelines for Haskell projects
This post collects a random assortment of guidelines I commonly share for how to organize Haskell projects.

Organize modules “vertically”, not “horizontally”
The glib summary of this rule is: don’t create a “Types” or “Constants” module.

“Vertically” organized modules are modules that group related functionality within the same module. For example, vertically-oriented modules for a simple interpreter might be:

A Syntax module

… which contains the concrete syntax tree type and utilities for traversing, viewing, or editing that tree.

A Parsing module

… which contains (or imports/re-exports) the parser type, a parser for the syntax tree, and error messages specific to parsing.

An Infer module

… which contains the the type inference logic, exception types, and error messages specific to type-checking.

An Evaluation module

… which logic for evaluating an expression, including possibly a separate data structure for a fully-evaluated abstract syntax tree.

A Pretty module

… which specifies how to pretty-print or otherwise format expressions for display.

“Horizontally” organized modules mean that you organize code into modules based on which language features or imports the code relies upon. For example, horizontally-oriented modules for the same interpreter might be:

A Types module

… which contains almost all types, including the concrete syntax tree, abstract syntax tree, parsing-related types, and exception types.

A Lib module

… which contains almost all functions, including the parsers, type inference code, evaluator, and prettyprinter.

A Constants module

… which contains almost all constants (including all error messages, timeouts, and help text).

An App module

… which contains the main entry point for the program.

There are a few reasons I prefer vertical module organization over horizontal module organization:

Vertically-organized modules are easier to split into smaller packages

For example, in a vertically-organized project I could separate out the Syntax, Parser, and Pretty modules into a separate standalone package, which could be used by other people to create an automatic code formatter for my language without having to depend on the type-checking or evaluation logic.

Conversely, there would be little benefit in separating out a Types module from the equivalent horizontally-organized package. Typically, horizontal modules are so tightly coupled to one another that no subset of the modules is useful in isolation.

The separability of vertical modules is an even bigger feature for proprietary projects that aspire to eventually open source their work. Vertically-organized projects are easier to open source a few modules at a time while keeping the proprietary pieces internal.

Vertically-organized modules tend to promote more granular and incremental build graphs

In a horizontally-organized project, each new type you add to the Types module forces a rebuild of the entire package. In a vertically-organized project, if I completely rewrite the type-checking logic then only other modules that depend on type-checking will rebuild (and typically very few depend on type-checking).

Vertically-organized modules tend to group related changes

A common issue in a horizontally-organized project is that every change touches almost every module, making new contributions harder and making related functionality more difficult to discover. In a vertically-organized project related changes tend to fall within the same module.

Naming conventions
I like to use the convention that the default module to import is the same as the package name, except replacing - with . and capitalizing words.

For example, if your package name is foo-bar-baz, then the default module the user imports to use your package is Foo.Bar.Baz.

Packages following this module naming convention typically do not have module names beginning with Control. or Data. prefixes (unless the package name happens to begin with a control- or data- prefix).

There are a few reasons I suggest this convention:

Users can easily infer which module to import from the package name

It tends to lead to shorter module names

For example, the prettyprinter package recently switched to this idiom, which changed the default import from Data.Text.Prettyprint.Doc to Prettyprinter.

It reduces module naming clashes between packages

The reasoning is that you are already claiming global namespace when naming a package, so should why not also globally reserve the module of the same name, too?

However, this won’t completely eliminate the potential for name clashes for other non-default modules that your package exports.

The “God” library stanza
This is a tip for proprietary projects only: put all of your project’s code into one giant library stanza in your .cabal file, including the entrypoint logic (like your command-line interface), tests, and benchmarks. Then every other stanza in the .cabal file (i.e. the executables, test suites, and benchmarks) should just be a thin wrapper around something exported from one of your “library” modules.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pete-murphy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
