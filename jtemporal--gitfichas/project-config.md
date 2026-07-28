---
trigger: always_on
description: - Always start the jekyll server with `bundle exec jekyll serve` no extra flags.
---

## Jekyll Development
- Always start the jekyll server with `bundle exec jekyll serve` no extra flags.

## Contributing Guidelines for GitFichas

When working on this GitFichas (Git Study Cards) project, follow these guidelines:

### New Mermaid Posts
- **Always run SVG generation** after creating or modifying posts with mermaid diagrams
- Use the command: `python3 scripts/generate_images_only.py`
- For specific posts: `python3 scripts/generate_images_only.py "filename.md"`
- Use `--force` flag to regenerate existing images: `python3 scripts/generate_images_only.py --force`
- Use `--verbose` flag for detailed debugging output when troubleshooting
- **Offer to run the script** when helping users create new mermaid posts
- Commit both the post changes AND the generated SVG files
- Generated SVG files are stored in `assets/img/mermaid/` directory

### Post Templates
Follow the templates from CONTRIBUTING.md exactly:

#### Command Cards Must Include:
- `layout: post` (mandatory)
- `pretitle` (optional) - for additional context before the title
- `title` (mandatory) - main card title
- `subtitle` (optional) - descriptive subtitle, can be blank
- `command` (mandatory) - the actual git command
- `descriptors` (mandatory) - descriptions for command parts
  - `command`: description of the main git command
  - `part1`, `part2`, etc.: descriptions for each command part
- `number` (mandatory) - card number with quotes for leading zeros
- `author` (mandatory) - GitHub username with @
- `mermaid: true` (mandatory)
- `use_static_image: true` (mandatory only when SVG image is generated)
- `permalink` (mandatory) - `/projects/{number}` for PT, `/en/{number}` for EN, `/es/{number}` for ES
- `lang` (mandatory) - "pt", "en", or "es"
- `pv` and `nt` (mandatory) - previous/next card navigation
- `{% include mermaid-graphs.html %}` at the end

#### Concept Cards Must Include:
- `layout: post` (mandatory)
- `pretitle` (optional) - for additional context before the title
- `title` (mandatory) - main concept title
- `subtitle` (optional) - can be blank
- `concept: true` (mandatory)
- `parts` (mandatory) - concept descriptions
  - `part1`, `part2`, etc.: individual concept explanations
- Same metadata as command cards (number, author, mermaid, etc.)

### Development Workflow
1. **Setup**: Recommend running `bash scripts/setup.sh` for new environments
   - This installs Node.js dependencies, Python packages, system libraries for headless Chrome, and Mermaid CLI
   - Auto-configures the entire development environment
2. **Create/Edit Posts**: Follow templates exactly
3. **Generate SVGs**: Always run `python3 scripts/generate_images_only.py`
4. **Test Locally**: Use `bundle exec jekyll serve` (no extra flags)
5. **Commit**: Include both post and generated SVG files

### Project Structure
- Posts with mermaid diagrams generate SVG files in `assets/img/mermaid/`
- Portuguese posts: `{number}.svg` (e.g., `053.svg`)
- English posts: `{number}-en.svg` (e.g., `053-en.svg`)
- Spanish posts: `{number}-es.svg` (e.g., `053-es.svg`)
- Theme configuration: `gitfichas-mermaid-theme.json`
- CSS files: `assets/css/mermaid.css`, `assets/css/embedded-svg.css`, `assets/css/embedded-fonts.css`

### Troubleshooting
- If SVG generation fails, ensure system dependencies are installed via `bash scripts/setup.sh`
- For browser launch errors, check that headless Chrome libraries are installed
- Missing fonts issue: run `python3 scripts/generate_embedded_fonts.py` first
- Use `--verbose` flag for detailed debugging information

### Card Numbering
- Use sequential numbering with leading zeros in quotes: `"053"`
- Check existing posts to find the next available number
- Update previous/next navigation links appropriately

### Languages
- Support multiple languages: PT (Portuguese), EN (English), and ES (Spanish)
- Use `lang: "pt"` for Portuguese cards in `_posts/`
- Use `lang: "en"` for English cards in `en/_posts/`
- Use `lang: "es"` for Spanish cards in `es/_posts/`
- Include `translations` array linking to other language versions when available
- Supported language codes: "pt", "en", "es"

### File Naming Convention
- **All files must follow Jekyll naming**: `YYYY-MM-DD-XXX.md`
- **For translations**: Use the SAME date as the original post
- Portuguese: `_posts/2024-10-26-054.md`
- English: `en/_posts/2024-10-26-054.md`
- Spanish: `es/_posts/2024-10-26-054.md`
- **Never change the date when creating translations**
- Check the original post filename to get the correct date

### Code Quality
- **Remove all trailing whitespaces** from files before committing
- Ensure consistent formatting and clean code standards

## Migrating cards from images to Mermaid

After migrating the content in the markdown files, you need to convert the existing image-based cards to the new Mermaid format.

**Important workflow:**

1. **Ensure proper file naming**: Use `YYYY-MM-DD-XXX.md` format with same date as original post for translations
2. **Generate SVG first**: Run image generation script one card at a time:
   ```bash
   python3 scripts/generate_images_only.py "filename.md"
   ```
3. **Add use_static_image only after successful SVG generation**: Add `use_static_image: true` to front matter ONLY after SVG is successfully created

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jtemporal/gitfichas](https://github.com/jtemporal/gitfichas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
