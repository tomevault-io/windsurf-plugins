---
trigger: always_on
description: Terminal presentations from Markdown using PowerShell 7.4+ and Spectre.Console.
---

# Deck Module Guidelines

Terminal presentations from Markdown using PowerShell 7.4+ and Spectre.Console.

## Code Style

- **OTBS** — opening brace on same line, `} else {`, `} catch {`, etc.
- **4-space indentation**, no tabs
- **CalVer** versioning: `YYYY.MM.DD.build`
- **Conventional commits**: `feat(scope):`, `fix(scope):`, `docs(scope):`, etc.
- **Comment-based help** on all functions with `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE` (3-5 examples), `.OUTPUTS`, `.NOTES`
- Cross-platform — no Windows-only assumptions

### Markdown Parsing Rules
Always skip parsing markup inside code:
- Replace inline backtick content with `___INLINECODE_N___` placeholders before processing bold/italic/color/strikethrough
- Restore placeholders after all formatting passes
- See [ConvertTo-SpectreMarkup.ps1](Private/ConvertTo-SpectreMarkup.ps1) for the canonical implementation

## Architecture

### Dependencies
- **PwshSpectreConsole** — rendering (tables, panels, aligned content, colors). Loaded at runtime via [Import-DeckDependency.ps1](Private/Import-DeckDependency.ps1); falls back to `Install-PSResource`, then sad ASCII art
- **TextMate** v0.1.0 — syntax highlighting for fenced code blocks via `Format-TextMate`. Declared in `RequiredModules`
- **Microsoft.PowerShell.PSResourceGet** 1.1.1

### Public API
- `Show-Deck` — the only exported cmdlet. Accepts `-Path` (file or URL), `-Strict` for validation, plus frontmatter overrides

### Slide Types (auto-detected)
| Type | Detection | Renderer |
|------|-----------|----------|
| Title | `#` heading only | [Show-TitleSlide.ps1](Private/Show-TitleSlide.ps1) |
| Section | `##` heading only | [Show-SectionSlide.ps1](Private/Show-SectionSlide.ps1) |
| Content | `###` heading + body | [Show-ContentSlide.ps1](Private/Show-ContentSlide.ps1) |
| Image | Content + `![alt](path)` | [Show-ImageSlide.ps1](Private/Show-ImageSlide.ps1) |
| Multi-Column | Content split by `\|\|\|` | [Show-MultiColumnSlide.ps1](Private/Show-MultiColumnSlide.ps1) |

### Key Private Functions
| Function | Purpose |
|----------|---------|
| [ConvertFrom-DeckMarkdown](Private/ConvertFrom-DeckMarkdown.ps1) | Parse YAML frontmatter + split slides |
| [ConvertTo-SpectreMarkup](Private/ConvertTo-SpectreMarkup.ps1) | Bold/italic/code/color → Spectre markup |
| [ConvertTo-CodeBlockSegments](Private/ConvertTo-CodeBlockSegments.ps1) | Split content into Text/Code/Table segments |
| [New-CodeBlockPanel](Private/New-CodeBlockPanel.ps1) | Fenced code → syntax-highlighted Spectre Panel via TextMate |
| [New-TableRenderable](Private/New-TableRenderable.ps1) | Markdown table → Spectre Table via Format-SpectreTable |
| [New-FigletText](Private/New-FigletText.ps1) | Heading → figlet renderable from bundled .flf fonts |
| [Get-SlideNavigation](Private/Get-SlideNavigation.ps1) | Keyboard input → navigation commands |
| [Get-TerminalDimensions](Private/Get-TerminalDimensions.ps1) | Terminal width/height detection |
| [Get-BorderStyleFromSettings](Private/Get-BorderStyleFromSettings.ps1) | Resolve border style enum |
| [Get-SpectreColorFromSettings](Private/Get-SpectreColorFromSettings.ps1) | Resolve color names/hex to Spectre colors |
| [Get-PaginationText](Private/Get-PaginationText.ps1) | Render pagination in configured style |
| [Import-DeckDependency](Private/Import-DeckDependency.ps1) | PwshSpectreConsole auto-install |
| [Show-SadFace](Private/Show-SadFace.ps1) | Dependency failure ASCII art |
| [Show-Logo](Private/Show-Logo.ps1) | Module logo display |

### Content Rendering Pipeline
1. `ConvertFrom-DeckMarkdown` parses frontmatter + splits into slides
2. Each slide dispatches to its type renderer
3. Content slides: inline formatting via `ConvertTo-SpectreMarkup`, code blocks via `New-CodeBlockPanel`, tables via `New-TableRenderable`
4. Multi-column slides: `ConvertTo-CodeBlockSegments` splits into Text/Code/Table segments per column
5. All renderables composed into full-height bordered panels with optional pagination

## Build and Test

```powershell
# import locally
Import-Module ./Deck.psd1

# run tests (Pester v5)
Invoke-Pester ./Tests/

# run a presentation
Show-Deck -Path ./Examples/ExampleDeck.md

# validate before presenting
Show-Deck -Path ./presentation.md -Strict
```

## Project Conventions

### New slide renderable pattern
When adding a new content type (like tables or code blocks):
1. Create a `New-*Renderable.ps1` in Private/ that returns `[Spectre.Console.IRenderable]`
2. Add detection regex to `Show-ContentSlide.ps1`, `Show-ImageSlide.ps1`, and `ConvertTo-CodeBlockSegments.ps1`
3. Dispatch in the segment rendering loop alongside existing Code/Table/Text branches
4. Add the file to `FileList` in `Deck.psd1`
5. Create a Pester test in Tests/ and a feature example in Examples/Features/

### Inline code placeholder pattern
`ConvertTo-SpectreMarkup` replaces backtick content with `___INLINECODE_N___` before bold/italic/color passes, then restores after. This prevents formatting regex from matching inside code spans. Any new inline formatting pass must run between the replace and restore steps.

### Table rendering

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jakehildreth/Deck](https://github.com/jakehildreth/Deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
