---
trigger: always_on
description: - [Warp Documentation Style Guide](#warp-documentation-style-guide)
---

# Table of Contents
- [Warp Documentation Style Guide](#warp-documentation-style-guide)
- [Warp Docs Repository Guide](#warp-docs-repository-guide)

# Warp Documentation Style Guide

This guide establishes standards for writing Warp documentation. It covers voice, formatting, content types, and terminology. Use it as the authoritative reference when creating or updating any page in the Astro Starlight repository.

## Writing style

### Voice & tone
- **Professional yet approachable**: Write with authority but remain accessible to developers of all skill levels
- **Direct and action-oriented**: Lead with what users can accomplish, not just what features exist
- **User-focused**: Use second person ("you can", "allows you to") rather than passive voice
- **Confident without jargon**: Explain technical concepts clearly without oversimplifying

### Language guidelines
- Use consistent terminology throughout (see [Terminology standards](#terminology-standards) and the full glossary in `.warp/references/terminology.md`)
- Em dashes are acceptable for occasional variation in narrative/conceptual text, but use sparingly
- Never use em dashes in procedural or instructional text

#### Active vs. passive voice
Use active voice whenever possible. Active voice is clearer and more direct.
- ✅ "Warp indexes your codebase to help Agents understand your code."
- ❌ "Your codebase is indexed by Warp to help Agents understand your code."

Passive voice is acceptable when the action's recipient is more important than the agent, or when the agent is unknown or irrelevant:
- ✅ "A critical security vulnerability was discovered in the authentication module." (emphasis on the vulnerability, not who found it)
- ✅ "The environment is destroyed after the run completes." (the system does this automatically; no human agent)

#### Ambiguous verbs
When a task is required, use clear, direct verbs. Avoid ambiguous modal verbs like "may," "might," "should," "could," "would," and "can" — these can be interpreted as either a command or a suggestion.
- ✅ "Use `oz agent run` to start a local agent." (required action)
- ✅ "You can optionally specify an Agent Profile." (clearly marked as optional)
- ❌ "You can use `oz agent run` to start a local agent." (is this required or optional?)
- ❌ "You should configure an environment before running cloud agents." (must I, or is it just a suggestion?)

#### Vague nouns and pronouns
If a pronoun could refer to more than one thing, replace it with the specific noun.
- ✅ "After you merge your pull request, you can delete the branch."
- ❌ "After you merge your pull request, you can delete it." (delete the PR or the branch?)

#### Stacked modifiers
Avoid strings of nouns that create ambiguity. Use prepositions to clarify relationships.
- ✅ "Default permissions for cloud agents"
- ❌ "Cloud agent default permission settings"

#### Nominalizations
Avoid turning verbs into nouns. Use the verb form for clearer, shorter sentences.
- ✅ "After the run completes, the container is destroyed."
- ❌ "After the completion of the run, the container undergoes destruction."

#### Invisible plurals
Avoid words that are ambiguous between singular and plural.
- ✅ "After the file is retrieved, select where to save it."
- ❌ "After file retrieval, select where to save it." (one file or many?)

### Punctuation and mechanics
- **Serial comma**: Always use it. "Environments, integrations, and schedules" — not "Environments, integrations and schedules."
- **Contractions**: Allowed and encouraged to match our approachable tone. Use "you're," "don't," "it's," "can't." Exception: avoid contractions in error messages or formal warnings.
- **Tense**: Use present tense to describe how things work ("Warp indexes your codebase"). Use imperative for instructions ("Configure your environment").
- **Person**: Use second person ("you") for instructions. Avoid first person plural ("we") in procedural content. First person is acceptable in conceptual or narrative text when referring to Warp as a company ("We designed Oz to...").

### Inclusive language
- Use gender-neutral pronouns ("they/them") for unknown users
- Avoid ableist language ("simple," "easy," "just" — these dismiss the reader's experience)
- Avoid culturally specific idioms or slang that may not translate across regions
- Describe UI elements by name and function, not by appearance alone (supports screen readers and non-visual contexts)

### Writing for accessibility and agents

These practices serve both human accessibility needs and AI agent consumption (AEO — Answer Engine Optimization).

**Accessibility:**
- Include captions or a brief text summary for video embeds so content is accessible without playing the video
- Don't rely on color alone to convey meaning (e.g., "the green status badge"). Always pair color with a text label (e.g., "the **Active** status badge")
- Use header rows in tables. Keep tables simple — avoid deeply nested structures
- Many rules in this guide (active voice, short sentences, plain language, descriptive links, alt text) also serve non-native English speakers and screen reader users

**Writing for agents (AEO):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [warpdotdev/docs](https://github.com/warpdotdev/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
