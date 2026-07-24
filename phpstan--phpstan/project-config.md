---
trigger: always_on
description: The PHPStan blog is written by Ondrej Mirtes, creator and maintainer of PHPStan. All articles are written in first person singular ("I"). The voice is conversational, technically authoritative, and occasionally humorous.
---

# PHPStan Blog Writing Style Guide

The PHPStan blog is written by Ondrej Mirtes, creator and maintainer of PHPStan. All articles are written in first person singular ("I"). The voice is conversational, technically authoritative, and occasionally humorous.

## Voice and Tone

- **First person**: Always "I", never "we" when referring to the author's decisions (though "we" is okay when referring to the PHPStan project or community collectively, e.g. "we improved BetterReflection").
- **Casual but knowledgeable**: Uses contractions ("it's", "doesn't", "I'm"), informal phrasing ("gonna", "dtto"), and direct address to the reader ("you").
- **Confident and opinionated**: Doesn't hedge or equivocate. States positions clearly. e.g. "I'd absolutely avoid inheritance", "the right solutions to both of these reasons are different, and I'm gonna tell you how and why."
- **Enthusiastic about static analysis**: Genuine excitement about features and improvements. e.g. "I've been looking forward to implementing and releasing the ideas present in PHPStan 1.10 for a long time."
- **Self-deprecating humor**: Occasional jokes at own expense. e.g. "I automated and scaled my favourite part of being a software developer: pointing out mistakes in other people's code", or footnotes like `[^lovemyjob]: I love my job!`
- **Emoji usage**: Very rare and restrained. Occasional use in informal contexts.
- **Explains the "why" before the "how"**: Motivates features by describing the problem they solve, often with real-world scenarios.

## Frontmatter

Every post has YAML frontmatter:

```yaml
---
title: "Title in Title Case With Quotes"
date: YYYY-MM-DD
tags: releases  # or: guides, other
---
```

Tags categorize articles:
- `releases` - Version announcements, new feature introductions
- `guides` - Explanations, tutorials, problem-solving articles
- `other` - Website meta, process, non-technical

## Article Types and Their Patterns

### Release Announcements (tag: releases)

Open with context about what makes this release special. Cover headline features with their own `##` sections. Each feature section explains the motivation, shows code examples, and links to documentation or playground. Often credits contributors by name with GitHub links. End with excitement about the future and the standard sponsorship CTA.

Example opening: "PHPStan 1.0 was released a little over three years ago. I'm happy to report the project is thriving! We did about 176 new releases since then..."

### Guides and Explanations (tag: guides)

Start by framing the problem or concept. Walk through solutions or explanations methodically. Heavy use of PHP code examples. Link to relevant documentation pages. Often structured as a series of solutions or approaches.

Example opening: "This error is reported for `new static()` calls that might break once the class is extended, and the constructor is overridden with different parameters."

### "Solving PHPStan error..." Series

Title format: `Solving PHPStan error "Exact error message here"`

These follow a consistent pattern:
1. Brief explanation of what triggers the error
2. Multiple solution sections, each with its own heading
3. Code examples showing the fix (often using `diff-php` syntax)
4. Links to relevant documentation

## Structure and Formatting

### Headings

Use either `##` (H2) or dashed-underline style:

```markdown
Section title
------------------------
```

Both styles are used; dashed underlines are more common in guides, `##` in release announcements.

### Code Examples

- PHP code in ` ```php ` fenced blocks
- NEON configuration in ` ```neon ` blocks
- Diff examples use ` ```diff-php ` with `+`/`-` prefixes
- Bash commands in ` ```bash ` blocks
- Comments inside code explain what PHPStan reports or what types are inferred
- Example: `// PHPStan reports: Unsafe usage of new static()`
- Example: `\PHPStan\dumpType($d); // MyDerivative :)`

### Footnotes

Used liberally for asides, jokes, and tangential notes:

```markdown
Something important. [^footnote]

[^footnote]: This is a tangential aside or joke.
```

Examples:
- `[^lovemyjob]: I love my job!`
- `[^parcel]: It might as well mean "I can't configure Webpack and I'm not ashamed!"`
- `[^telemetry]: This is just from my personal experience and anecdotal evidence - PHPStan doesn't perform any telemetry on user's code.`

### Links

- Cross-references to other blog posts: `[Learn more](/blog/article-slug)`
- Links to documentation: `[configuration](/config-reference)`
- Playground examples: `[playground example](/r/uuid-here)` or `[Link to this example on the playground](/r/uuid-here)`
- GitHub releases: `[PHPStan 1.9.0](https://github.com/phpstan/phpstan/releases/tag/1.9.0)`
- Credit contributors with GitHub profile links

### Embedded Tweets

Used as social proof, community reactions, or to reference prior statements. Full HTML `<blockquote class="twitter-tweet">` embeds with the Twitter widgets script.

### Markdown Abbreviations

Occasionally used for technical acronyms:

```markdown
*[AST]: Abstract Syntax Tree
*[OOP]: Object-Oriented Programming
*[TLC]: Tender Loving Care
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phpstan/phpstan](https://github.com/phpstan/phpstan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
