---
trigger: always_on
description: Documentation and content creation standards
---


## Markdown Content Rules

The following markdown content rules are enforced in the validators:

1. **Headings**: Use appropriate heading levels (H2, H3, etc.) to structure your content. Do not use an H1 heading, as this will be generated based on the title.
2. **Lists**: Use bullet points or numbered lists for lists. Ensure proper indentation and spacing.
3. **Code Blocks**: Use fenced code blocks for code snippets. Specify the language for syntax highlighting.
4. **Links**: Use proper markdown syntax for links. Ensure that links are valid and accessible.
5. **Images**: Use proper markdown syntax for images. Include alt text for accessibility.
6. **Tables**: Use markdown tables for tabular data. Ensure proper formatting and alignment.
7. **Line Length**: Limit line length to 400 characters for readability.
8. **Whitespace**: Use appropriate whitespace to separate sections and improve readability.
9. **Front Matter**: Include YAML front matter at the beginning of the file with required metadata fields.

## Formatting and Structure

Follow these guidelines for formatting and structuring your markdown content:

- **Headings**: Use `##` for H2 and `###` for H3. Ensure that headings are used in a hierarchical manner. Recommend restructuring if content includes H4, and more strongly recommend for H5.
- **Lists**: Use `-` for bullet points and `1.` for numbered lists. Indent nested lists with two spaces.
- **Code Blocks**: Use triple backticks to create fenced code blocks. Specify the language after the opening backticks for syntax highlighting (e.g., `csharp`).
- **Links**: Use `[link text](URL)` for links. Ensure that the link text is descriptive and the URL is valid.
- **Images**: Use `![alt text](image URL)` for images. Include a brief description of the image in the alt text.
- **Tables**: Use `|` to create tables. Ensure that columns are properly aligned and headers are included.
- **Line Length**: Break lines at 80 characters to improve readability. Use soft line breaks for long paragraphs.
- **Whitespace**: Use blank lines to separate sections and improve readability. Avoid excessive whitespace.

## Follow our Guidelines

### Spelling

In cases where American spelling differs from Commonwealth/"British" spelling, use the American spelling.

Although non-American readers tend to be tolerant of reading American spelling in technical documentation,
they may find it difficult to have to type American spelling.
For example, if your documentation tells a reader who's used to the spelling colour to type color,
they may mistype it. So when you use filenames, URLs, and data parameters in examples,
try to avoid words that are spelled differently by different groups of English speakers.

### Write accessibly

#### Ease of reading

* Do not force line breaks (hard returns) within sentences and paragraphs.
  Line breaks might not work well in resized windows or with enlarged text.
* Break up walls of text to aid in scannability.
  For example, separate paragraphs, create headings, and use lists.
* Prefer short sentences.
* Define acronyms and abbreviations on first usage and if they are used infrequently.
* Place distinguishing and important information of a paragraph in the first sentence to aid in scannability.
* Use clear and direct language. Avoid the use of double negatives and exceptions in exceptions.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```markdown
A missing path will not prevent you from continuing.
```

<ul>
  <li>Double negation (missing, not)</li>
  <li>Use of future tense (will)</li>
</ul>
</td><td>

```markdown
You can continue without a path.
```

</td></tr>
</tbody></table>

#### Headings and titles

Use descriptive headings and titles because they help a reader navigate their browser and the page.
It's easier to jump between pages and sections of a page if the headings and titles are unique.

* Use a heading hierarchy.
* Do not skip levels of hierarchy (`h3` can only exist under `h2`)
* Do not use empty headings
* Use a level-1 heading for the page title.
* Use sentence casing for titles and headings.

#### Links

* Use meaningful link text. Links should make sense when read out of context.
* Do not force links to open in a new tab or window, let the reader decide how to open links.
* When possible, avoid adjacent links. Instead, put at least one character in between to separate them.
* If a link downloads a file, indicate this action and the file type in the link text.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```markdown
Use meaningful link text like described [here](https://developers.google.com/style/link-text).
Use meaningful link text. [See document.](https://developers.google.com/style/link-text)
Use meaningful link text. https://developers.google.com/style/link-text
```

</td><td>

```markdown
Use [meaningful link text](https://developers.google.com/style/link-text).
```

</td></tr>
</tbody></table>

#### Images

* When possible, use SVG images over any other format, since they are significantly lighter while having perfect information.
* For every image, provide alt text that adequately summarizes the intent of each image.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ullaakut/cameradar](https://github.com/Ullaakut/cameradar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
