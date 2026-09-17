---
trigger: always_on
description: elemctl is a public, international project: a CLI, an MCP server and a Python library for the
---

# Repository conventions

elemctl is a public, international project: a CLI, an MCP server and a Python library for the
1C:Enterprise.Element Console API v2. The contract the tool implements lives in
[docs/SPEC.md](docs/SPEC.md), with a Russian twin in `docs/SPEC.ru.md`. This file records how
the repository itself is written.

## Language of the code

- **Code is English.** Comments, docstrings, identifiers, test names – all of them, in `src/`,
  `tests/`, `tools/` and `editors/`. The code is read by people who do not speak Russian.
- **Russian stays where it faces the user**: the i18n message catalog (`src/elemctl/i18n.py`),
  argparse help strings, user-facing strings, the MCP tool descriptions and the server
  `INSTRUCTIONS` literal. An agent reads those in Russian.
- **Platform identifiers are quoted as they are**: `Проект.yaml`, `Ресурсы`, `Имя`, `Поставщик`,
  `ВидПроекта`, `ОбластьВидимости` and the like are real keys and file names, so do not
  translate them. The same goes for the Russian data of test fixtures.

## Typography

Applies to English text as well:

- dashes – en dash `–` (U+2013) only, never an em dash;
- quotes – straight `"` and `'`, never guillemets or curly quotes;
- ellipsis – three dots `...`, never the `…` character.

## Russian without borrowed words

The Russian edition kept drifting into English written in Cyrillic letters. An entry said that
a `пин` had been raised after a `прогон`, and the reader had to translate both before the
sentence meant anything. The words and the Russian to write instead live in the `docsguard`
package. This repository only says which of its documents are Russian, in
`scripts/check_docs.py`: the pages under `docs/` are found by pattern, the README, the changelog
and this file by name.

A word quoted as a word goes in backticks. `пин` inside them is a name being discussed rather
than a word being used, and the guard reads it as a name, the way it reads a fenced block, a
link target and a file name. The caption of a command is written the same way, whoever the
command belongs to.

## Nothing internal

The repository is public. It must not carry internal project identifiers, stand names, real
application or assembly ids, internal hosts, issue keys or machine paths. That holds for the
code, for comments and for test fixtures alike. Use neutral examples: vendors `acme` and
`globex`, applications `crm-dev` and `demo-app`.

A text also says what was wrong, never who asked for the change. Reviews and decisions happen
off the page, and the person who writes the code here is the person who owns it, so a line
about an owner asking for something reads as if there were someone else above the author.
"The sentences ran to five lines and the words read as transliteration" says the same thing and
survives being read by a stranger.

## Documentation pairs

English and Russian pages go together: `README.md` / `README.ru.md`, `docs/SPEC.md` /
`docs/SPEC.ru.md`, `CHANGELOG.md` / `CHANGELOG.ru.md` and the rest of `docs/*.md`. A change
to one side without the other is an unfinished change. Four pages are generated, so never edit
them by hand:

- `docs/cli.md` / `docs/cli.ru.md` – from the output of `elemctl ... --help`;
- `docs/changelog.md` / `docs/changelog.ru.md` – from the root `CHANGELOG` editions.

One command rebuilds all of them: `python scripts/rebuild-docs.py`. It is the only one to
remember. Every generator used to carry a command of its own, and each of those turned into
something else to remember. The mirrors were the first to be left behind, and `main` went red
on the guard for it.

The pull request link that every changelog entry ends with is written by
`python scripts/changelog-link.py <number>` rather than by hand. It appends the link to every
entry of the topmost section in both editions and rebuilds the generated pages in the same run.
The link and the rebuild are two halves of one step, and doing the first by hand is how the
second gets forgotten.

## One fact, one wording

A statement about the platform is told in several places at once: the specification, the
Console API page, the MCP page, the README, the docstrings of the code. Then it gets corrected
in one of them. Twice that left the rest telling the model it replaced, and one document ended
up carrying both at the same time. Such statements are listed as `CLAIMS` in
`scripts/check_docs.py`. A row names the places that must state the fact, the spellings that
count as stating it, and the spellings of the model it replaced. Those old spellings may appear
nowhere but the changelog, where an entry about a correction quotes what it corrected.
Correcting such a fact means correcting every place the claim names, and the guard says which
one was missed. A fact that starts living in more than one place gets a row of its own. The
judging itself comes from the shared `docsguard` package, because the neighbouring repositories
keep their documentation the same way and have the same defect waiting. What lives here is the
table.

## The shared guard

The parts of the documentation guard that three repositories were keeping in triplicate live in
the `docsguard` package, which the workflows install from git. It is fixed to a tag, never to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keyfire/elemctl](https://github.com/keyfire/elemctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
