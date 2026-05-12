---
trigger: always_on
description: **ALWAYS follow these instructions first and fallback to additional search and context gathering only when the information here is incomplete or found to be in error.**
---

# Goyo - Zola Theme Development Instructions

**ALWAYS follow these instructions first and fallback to additional search and context gathering only when the information here is incomplete or found to be in error.**

Goyo is a Zola theme for static site generation that creates clean, minimalist documentation sites. This repository serves both as the theme itself and as a documentation/demo site showcasing the theme's capabilities. Inspired by the Korean word "Goyo" (고요), meaning calm or serene, the theme pursues minimalism and simplicity.

## Working Effectively

### Prerequisites Installation
Install required tools in this exact order:

```bash
# Install Zola (static site generator)
cd /tmp
wget https://github.com/getzola/zola/releases/download/v0.21.0/zola-v0.21.0-x86_64-unknown-linux-gnu.tar.gz
tar -xzf zola-v0.21.0-x86_64-unknown-linux-gnu.tar.gz
sudo mv zola /usr/local/bin/
zola --version  # Should show: zola 0.21.0

# Install Just (task runner)
wget https://github.com/casey/just/releases/download/1.42.4/just-1.42.4-x86_64-unknown-linux-musl.tar.gz
tar -xzf just-1.42.4-x86_64-unknown-linux-musl.tar.gz
sudo mv just /usr/local/bin/
just --version  # Should show: just 1.42.4
```

### Project Setup
Bootstrap the project dependencies:

```bash
# Setup dependencies for Linux (x86_64)
just setup-linux

# OR Setup dependencies for macOS (Apple Silicon)
# just setup-macos

# Verify TailwindCSS works
src/tailwindcss --help
```

**NOTE:** `just setup-macos` downloads the ARM64 version of TailwindCSS. For Intel Macs or other architectures, you may need to download the correct binary manually to `src/tailwindcss`.

### Build Process
Build the complete site:

```bash
just build
```

**TIMING:** Build completes in ~0.5 seconds. NEVER CANCEL builds - they complete quickly.

This command:
1. Compiles CSS with TailwindCSS (~190ms)
2. Builds the static site with Zola (~140ms)
3. Creates 32 pages and 14 sections
4. Generates multilingual content (English + Korean)

### Development Server
Start the development server:

```bash
just dev
```

This runs the build process then starts the Zola development server at `http://127.0.0.1:1111`. The server automatically rebuilds on file changes.

**TIMING:** Server startup takes ~0.5 seconds. NEVER CANCEL - it starts quickly.

### Validation Commands
Always run these validation steps after making changes:

```bash
# Check internal links and site structure (SKIP external link check due to network restrictions)
zola check --skip-external-links

# Clean build test
rm -rf public && just build

# Verify dev server works
just dev  # Then test http://127.0.0.1:1111 in another terminal
```

## Manual Validation Requirements

After making any changes to the theme or content:

1. **Build Validation**: Always run a clean build and verify it completes without errors
2. **Server Test**: Start the dev server and verify the homepage loads at `http://127.0.0.1:1111`
3. **Content Check**: Verify key pages render correctly:
   - Landing page: `http://127.0.0.1:1111` (should show "Welcome to Goyo!")
   - Documentation: `http://127.0.0.1:1111/introduction/`
   - Korean version: `http://127.0.0.1:1111/ko/`
4. **Theme Features**: Verify core theme functionality works:
   - Dark/light mode toggle
   - Navigation menu
   - Sidebar navigation
   - Search functionality

## Common Tasks and Navigation

### Repository Structure
```
/home/runner/work/goyo/goyo/
├── .github/
│   ├── workflows/           # GitHub Actions deployment (zola.yml, labeler.yml)
│   ├── labeler.yml          # GitHub labeler configuration
│   └── FUNDING.yml          # Sponsorship configuration
├── AGENTS.md                # Development instructions for AI assistants
├── content/                 # Markdown content files (multilingual: .md for EN, .ko.md for KR)
│   ├── _index.md            # Landing page configuration
│   ├── introduction/        # Introduction section
│   ├── get_started/         # Getting started guides (installation, configuration, creating_content, creating_landing)
│   ├── writing/             # Writing guides (markdown-syntax, shortcodes)
│   ├── references/          # Reference documentation
│   ├── deployments/         # Deployment guides (github_pages, Other)
│   └── contributing/        # Contribution guidelines
├── static/                  # Static assets served directly
│   ├── css/                 # Compiled CSS (main.css, font-awesome.min.css, katex.min.css)
│   ├── js/                  # JavaScript files (copy_code.js, copy_heading_link.js, katex.min.js, mermaid.min.js)
│   ├── fonts/               # Font files
│   ├── icons/               # Favicon and app icons
│   ├── images/              # Image assets (logo, landing images, thumbnails)
│   ├── resources/           # External resources (zola.svg, tailwindcss.svg, daisyui.svg)
│   ├── goyo.js              # Main theme JavaScript
│   └── fuse.min.js          # Search library
├── templates/               # Zola HTML templates (Tera templating engine)
│   ├── index.html           # Default index template
│   ├── page.html            # Page template
│   ├── section.html         # Section template

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hahwul/goyo](https://github.com/hahwul/goyo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
