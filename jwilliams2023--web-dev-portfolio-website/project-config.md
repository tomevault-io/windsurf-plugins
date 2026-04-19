---
trigger: always_on
description: This is a personal portfolio website for Joseph Williams, a Master's student in Computer Science at Auburn University. The site showcases his experience, projects, achievements, and skills in Software Engineering, Data Science, and AI/ML.
---

# LLM Context for Joseph Williams' Portfolio Website

## Project Overview
This is a personal portfolio website for Joseph Williams, a Master's student in Computer Science at Auburn University. The site showcases his experience, projects, achievements, and skills in Software Engineering, Data Science, and AI/ML.

## Tech Stack
- **Frontend**: HTML5, CSS3 (with custom theme system), JavaScript (vanilla)
- **Build Tool**: Vite
- **Deployment**: Netlify
- **API Integration**: Hugging Face API for AI summarization feature via Netlify serverless function

## Key Features
1. **Dark/Light Theme System**: Auto-detects system preferences with manual toggle option
2. **AI Summarization**: Uses Hugging Face API to summarize the "About Me" section
3. **Responsive Design**: Mobile-first approach with clean, professional styling
4. **Expandable Sections**: "View More" toggles for About, Experience, and Achievements
5. **Version Display**: Auto-updates version from package.json in footer

## Project Structure
- `/index.html` - Main HTML file with all content sections
- `/src/style.css` - All styling with CSS variables for theming
- `/src/main.js` - JavaScript for theme toggling, expandable sections, and AI features
- `/netlify/functions/summarize.js` - Serverless function for AI summarization
- `/public/` - Static assets (resume PDFs, images, etc.)
- `vite.config.js` - Vite build configuration
- `netlify.toml` - Netlify deployment configuration

## Sections
1. **Navigation**: Sticky navbar with links to all sections
2. **Introduction/Hero**: Name, tagline, social links
3. **About Me**: Bio, skills, tools, AI summarization feature
4. **Experience**: Work experience and research positions (expandable)
5. **Projects**: Technical projects with links and descriptions
6. **Achievements**: Scholarships, awards, certifications, memberships (expandable)
7. **Contact**: Email and social media links
8. **Footer**: Credits and version number

## Design System
- **Primary Color**: Bright blue (#384bf7)
- **Theme Variables**: CSS custom properties for light/dark mode
- **Typography**: Clean, readable fonts with consistent sizing
- **Spacing**: Consistent padding and margins using rem units
- **Cards**: Unified styling with max-width, shadows, and hover effects

## Development Notes
- The site follows semantic HTML practices
- All icons are from CDN sources (devicons, Wikipedia)
- Theme persistence uses localStorage
- Version number is dynamically injected from package.json during build
- Left-aligned text and headings for consistency across all sections

## Change History (Changelog)

All notable changes to this project are documented below.

**Note:** This changelog was formalized after initial development. Earlier entries are inferred from git commit history.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

### [Unreleased]

### [1.2.0] - 2026-02-25
#### Added
- ICPC 2025 USA Southeast Division 1 Regional 4th place achievement
- ColorStack organization membership (August 2024 - Present)
- Certifications section with AI/ML Engineering Certificate and Databricks credentials
- AI/ML Engineering Certificate mention in About Me section
- Handshake AI Fellow experience (September 2025 - February 2026)
- USAA Decision Science Intern position (May 2026 - August 2026)
- SVG icons for theme toggle (moon and sun) replacing emoji
- Validation link for Auburn AI/ML Engineering Certificate

#### Changed
- Updated About Me to reflect Master's student status at Auburn University
- Mentioned AI/ML Engineering Certificate alongside Magna Cum Laude
- Reordered sections: Experience now appears before Projects
- Left-aligned all section headings (h2, h3) and body text for consistency
- Updated all experience dates to full month names (e.g., "June" instead of "Jun")
- Consolidated Auburn research positions into single entry (August 2023 - December 2025)
- Added Computer Vision research under Dr. Brian Thurow to research experience
- Updated ACM membership end date to 2025 (from Present)
- Changed Competitive Programming Team role from "Active member" to "Competitor"
- Moved theme toggle button to vertical center of screen (right side)
- Simplified footer to only show "Back to Top" link
- Updated resume link to 2026 version
- Email address updated to @proton.mail domain
- Removed max-width constraints on experience and achievement cards for better alignment

#### Fixed
- Fixed footer buttons stacking issue - now properly displays inline
- Removed empty space between Business & Innovation and Contact sections
- Fixed alignment inconsistencies across Achievements section
- Fixed resume button path to correctly point to 2026 resume file

#### Removed
- Removed "Quick Links" navigation from footer
- Removed duplicate Licenses & Certifications section
- Removed Remote/Hybrid location designations from experience entries
- Removed ICPC competition mentions from ACM description (redundant with dedicated ICPC entries)

### [1.1.2] - 2025-01-27
#### Added

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jwilliams2023) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
