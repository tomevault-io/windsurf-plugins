---
trigger: always_on
description: > This page is to quickly check how all features are rendered.
---


> This page is to quickly check how all features are rendered.

- Theme author: <a href="https://github.com/arun-gajaraj" target="_blank" rel="noopener noreferrer">Arun G</a>
- Support the author: Author did not provide funding links in `manifest.json`
- <a href="https://github.com/saberzero1/quartz-themes/tree/master/themes/claude/README.md">Install instructions</a>

| Name     | Modes                     | Compatibility                                  | License                                                                                                                                       |
| -------- | ------------------------- | ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `claude` | ![[media/dark.svg\|DARK]] | <img src="media/checking.svg" alt="CHECKING"/> | <a href="https://github.com/saberzero1/quartz-themes/tree/master/obsidian/claude/LICENSE.md"><img src="media/license/mit.svg" alt="MIT"/></a> |

# Markdown Formatting Guide for Obsidian

Learn how to apply basic formatting to your Obsidian notes, using [Markdown](https://daringfireball.net/projects/markdown/). Import this as a note into your Obsidian Vault to view it and to test new themes. Certain community plugins are required for a few features.

---

# Basic Formatting Syntax

---

## Paragraphs

To create paragraphs, use a blank line to separate one or more lines of text.

```
This is a paragraph.

This is another paragraph.
```

Multiple blank spaces

Multiple adjacent blank spaces in and between paragraphs collapse to a single space when displaying a note in [Reading view](https://help.obsidian.md/Editing+and+formatting/Editing+and+previewing+Markdown#Editor%20views) and on [Obsidian Publish](https://help.obsidian.md/Obsidian+Publish/Introduction+to+Obsidian+Publish) sites.

```md
Multiple adjacent spaces

and multiple newlines between paragraphs.
```

> Multiple adjacent spaces
>
> and multiple newlines between paragraphs.

If you want to add multiple spaces, you can add `&nbsp;` (blank space) and `<br>` (newline) to your note.

```md
Multiple&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;adjacent&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spaces
<br>
<br>
<br>
and multiple newlines between paragraphs.
```

> Multiple&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;adjacent&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spaces
> <br>
> <br>
> <br>
> and multiple newlines between paragraphs.

## Headings

To create a heading, add up to six `#` symbols before your heading text. The number of `#` symbols determines the size of the heading.

```md
# This is a heading 1

## This is a heading 2

### This is a heading 3

#### This is a heading 4

##### This is a heading 5

###### This is a heading 6
```

# This is a heading 1

## This is a heading 2

### This is a heading 3

#### This is a heading 4

##### This is a heading 5

###### This is a heading 6

## Styling text

| Style                  | Syntax                 | Example                                  | Output                                 |
| ---------------------- | ---------------------- | ---------------------------------------- | -------------------------------------- |
| Bold                   | `** **` or `__ __`     | `**Bold text**`                          | **Bold text**                          |
| Italic                 | `* *` or `_ _`         | `*Italic text*`                          | _Italic text_                          |
| Strikethrough          | `~~ ~~`                | `~~Striked out text~~`                   | ~~Striked out text~~                   |
| Highlight              | `== ==`                | `==Highlighted text==`                   | ==Highlighted text==                   |
| Bold and nested italic | `** **` and `_ _`      | `**Bold text and _nested italic_ text**` | **Bold text and _nested italic_ text** |
| Bold and italic        | `*** ***` or `___ ___` | `***Bold and italic text***`             | **_Bold and italic text_**             |

## Quotes

You can quote text by adding a `>` symbols before the text.

```md
> Human beings face ever more complex and urgent problems, and their effectiveness in dealing with these problems is a matter that is critical to the stability and continued progress of society.

\- Doug Engelbart, 1961
```

> Human beings face ever more complex and urgent problems, and their effectiveness in dealing with these problems is a matter that is critical to the stability and continued progress of society.

- Doug Engelbart, 1961

> [!info]
> You can turn your quote into a [callout](https://help.obsidian.md/Editing+and+formatting/Callouts) by adding `[!info]` as the first line in a quote.

## Callouts

> [!info]
> Default title

> [!question]+ Can callouts be _nested_?
>
> > [!todo]- Yes!, they can. And collapsed!
> >
> > > [!example] You can even use multiple layers of nesting.

> [!note]
> Aliases: "note"

> [!abstract]
> Aliases: "abstract", "summary", "tldr"

> [!info]
> Aliases: "info"

> [!todo]
> Aliases: "todo"

> [!tip]
> Aliases: "tip", "hint", "important"

> [!success]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saberzero1/quartz-themes](https://github.com/saberzero1/quartz-themes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
