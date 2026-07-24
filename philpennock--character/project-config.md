---
trigger: always_on
description: This file describes the `character` CLI tool for the benefit of AI coding
---

# AGENTS.md — Guide for AI Coding Agents

This file describes the `character` CLI tool for the benefit of AI coding
assistants, LLM-based agents, and automated tooling.  The tool answers
questions about Unicode codepoints, character properties, emoji, and related
transformations.

For understanding the **source code** — package structure, data flow, reading
order, protocols, and architectural invariants — see
[`CODE_GUIDE.md`](CODE_GUIDE.md).


## What this tool does

`character` is a Unicode character-information CLI.  Given a character, a
name, or a codepoint number, it returns authoritative Unicode metadata:
official name, hex codepoint, UTF-8 encoding bytes, block membership, HTML/XML
entity aliases, Vim digraphs, and more.

Common uses for an AI agent:
- Look up the canonical name of a character so you can refer to it precisely.
- Find a character by name or substring when you know what it looks like but
  not its codepoint.
- Verify the exact bytes that will appear in source code when using a Unicode
  literal.
- Look up emoji codepoints and their modifier sequences before writing them
  into code, comments, or user-facing strings.
- Get the right representation (text vs. emoji presentation variant) for a
  character.


## Quick-reference: key sub-commands

| Sub-command | Purpose |
|---|---|
| `agent help` | JSON schema of all commands and flags (agent-oriented) |
| `agent examples` | Runnable shell examples for common agent use-cases |
| `agent mcp` | Start an MCP server (stdio) exposing Unicode lookups as tools |
| `name <char>…` | Properties of literal character(s) |
| `named <NAME>` | Look up a character by its exact Unicode name |
| `named -j <MULTI WORD NAME>` | Join all args as one name |
| `named -/ <word>` | Substring search across all character names |
| `search <word>` | Alias for `named -v/` (verbose substring search, table) |
| `code <U+XXXX>` or `code <decimal>` | Look up by codepoint |
| `browse -b <block-name>` | List all characters in a Unicode block |
| `browse -f <U+X> -t <U+Y>` | List characters in a codepoint range |
| `transform fraktur <text>` | Convert text to Fraktur mathematical letters |
| `transform math <text>` | Mathematical letter variants |
| `transform scream <text>` | Reversible "scream" encoding |
| `transform turn <text>` | Upside-down character transformation |
| `region <XY>` | Convert two-letter country code to flag emoji regional indicators |
| `x-puny <string>` | Punycode encode/decode |
| `known -b` | List all Unicode block names |
| `known -e` | List supported character-set encoding names |
| `version -j` | Version info as JSON |


## Output formats

### Plain output and the one-line contract

For **`named`**, **`code`**, and **`named -/`** (search): plain output is
exactly one line per result — either the character glyph (default for `named`
and `code`) or the character name.  This contract is strict and reliable for
programmatic use.

For **`name`**: the default is one line per input rune, but two categories of
input cause additional lines to be emitted:

- **Variation selectors** (U+FE00–U+FE0F, U+E0100–U+E01EF): the selector
  itself gets a line, and a second extra line shows the base character and
  selector combined.  So `character name ☃︎` (U+2603 + U+FE0E) emits three
  lines: `SNOWMAN`, `VARIATION SELECTOR-15`, then the combined glyph.

- **Regional indicator pairs** (U+1F1E6–U+1F1FF): each indicator gets a line,
  and a third line summarises the completed pair.  So `character name 🇫🇷`
  emits three lines for the two input codepoints.

Use `-J` / `--json` to avoid any line-count ambiguity in all commands.

### Flags that change output format

| Flag | Effect |
|---|---|
| `-v` | Verbose: render a rich table with all default columns |
| `-N` | Net-verbose: adds network-oriented columns (IDN, punycode) |
| `--json` / `-J` | JSON output — **preferred for programmatic/agent use** |
| `-1` / `--oneline` | All characters on a single line (not available on `name`) |
| `-c` / `--clipboard` | Copy output characters to clipboard (interactive use) |

### JSON output

The `-J` / `--json` flag is the most reliable format for agent consumption.
It removes terminal-rendering concerns (box-drawing characters, column widths)
and gives well-typed fields.  Example:

```
$ character named -Jj SNOWMAN
```

```json
{
  "characters": [
    {
      "display": "☃",
      "name": "SNOWMAN",
      "hex": "2603",
      "decimal": "9731",
      "utf8": "%E2%98%83",
      "block": "Miscellaneous Symbols",
      "htmlEntities": ["snowman"],
      "vimDigraphs": ["sn"],
      "jsonEscape": "\\u2603",
      "renderWidth": 1
    }
  ]
}
```

Key fields in JSON output:

| Field | Content |
|---|---|
| `display` | The character glyph |
| `displayText` | Glyph with text presentation selector appended |
| `displayEmoji` | Glyph with emoji presentation selector appended |
| `name` | Official Unicode name |
| `hex` | Codepoint in hexadecimal (no prefix) |
| `decimal` | Codepoint as decimal string |
| `utf8` | UTF-8 bytes in URL-percent encoding (`%XX%YY%ZZ`) |
| `jsonEscape` | Ready-to-use JSON/JavaScript escape (`\uXXXX` or surrogate pair) |
| `block` | Unicode block name |
| `vimDigraphs` | Vim digraph sequences, if any |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [philpennock/character](https://github.com/philpennock/character) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
