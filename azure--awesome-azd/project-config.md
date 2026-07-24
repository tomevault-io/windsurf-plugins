---
trigger: always_on
description: Awesome azd is a Docusaurus-based website showcasing Azure Developer CLI templates. The site serves as a discovery platform for azd templates, provides contribution guides, and hosts a searchable gallery of community and Microsoft-authored templates.
---

# Awesome Azure Developer CLI (azd) 

Awesome azd is a Docusaurus-based website showcasing Azure Developer CLI templates. The site serves as a discovery platform for azd templates, provides contribution guides, and hosts a searchable gallery of community and Microsoft-authored templates.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

Bootstrap, build, and test the repository:

- Ensure Node.js 18+ is available: `node --version` (should show v20.19.5 or later)
- Navigate to website directory: `cd website`
- Install dependencies: `npm ci` -- takes ~15-60 seconds with warnings about deprecated packages (this is normal)
- Run tests: `npm test` -- takes ~1 second. NEVER CANCEL. Set timeout to 30+ seconds.
- Build the website: `npm run build` -- takes ~75 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
- Start development server: `npm run start` -- takes ~30 seconds to compile. NEVER CANCEL. Set timeout to 60+ seconds.
- Serve production build: `npm run serve` -- starts immediately after build

## Critical Timing and Timeout Requirements

- **npm ci**: 15-60 seconds (normal with deprecated package warnings about rimraf, glob, inflight)
- **npm test**: 1 second (set timeout to 30+ seconds)
- **npm run build**: 75 seconds (set timeout to 120+ seconds)  
- **npm run start**: 30 seconds initial compile (set timeout to 60+ seconds)
- **NEVER CANCEL any build or test commands** - they will complete successfully

## Manual Validation Requirements

After making any changes to the website, ALWAYS validate functionality by:

1. Start the development server: `cd website && npm run start`
2. Wait for "Compiled successfully" message
3. Test website accessibility: `curl -f "http://localhost:3000/awesome-azd/"`
4. Navigate to key pages and verify:
   - Homepage displays template gallery correctly
   - Getting Started page loads with video and action cards
   - Contribute page shows documentation
   - All navigation links work

## Repository Structure

Key directories and files:
- `website/` - Main Docusaurus application
  - `src/` - React components, pages, and data
  - `docs/` - Markdown documentation files
  - `static/` - Static assets including `templates.json` (5548 lines)
  - `package.json` - Dependencies and npm scripts
- `.github/workflows/` - CI/CD pipelines (release.yml, test-deploy.yml)
- `README.md` - Project overview and resources
- `GALLERY.md` - Gallery documentation

## Available Commands

All commands must be run from the `website/` directory:

- `npm ci` - Install dependencies (required first step)
- `npm test` - Run Jest tests (validates templates.json against tags)
- `npm run build` - Build production website to `build/` directory
- `npm run start` - Start development server at http://localhost:3000/awesome-azd/
- `npm run serve` - Serve production build locally
- `npm run docusaurus` - Direct Docusaurus CLI access

## Key Technologies

- **Docusaurus 3.7.0** - Static site generator
- **React 18** - UI framework
- **TypeScript** - Type checking
- **Jest** - Testing framework
- **Fluent UI** - Microsoft design system components
- **Node.js 20+** - Runtime requirement

## Template Gallery System

The template gallery is driven by:
- `website/static/templates.json` - Template metadata (5548 lines)
- `website/src/data/tags.tsx` - Tag definitions for filtering
- `website/src/components/gallery/` - Gallery components
- `website/static/templates/images/` - Template preview images

### Template Requirements

Templates must meet these standards to be included in the collection:

**README Requirements:**
- Standard structure with project name, use case, features, and architecture diagram
- Step-by-step deployment and customization instructions
- Getting started section with quick setup options (GitHub Codespaces, Dev Containers, local)
- Clear feature list highlighting AI capabilities
- CI/CD pipeline setup guidance using azd commands

**Security Recommended Practices:**
- Use keyless authentication (Managed Identity preferred or Key Vault) instead of API keys
- Implement Role-Based Access Control (RBAC) for resource access
- Enable data encryption at rest and in transit
- Use Azure Key Vault for secrets management
- Configure network isolation with private endpoints where applicable
- Include monitoring with Azure Monitor and Application Insights
- Add Responsible AI governance controls (content safety filters, etc.)
- Include SECURITY.md file with security reporting procedures

**azure.yaml Metadata Requirements:**
- **name** field (required): Unique app/template name using lowercase, numbers, dashes
- **metadata.template** field (recommended): Template identifier with version (e.g., `app-name@1.0.0`)
- List all Azure services and dependencies
- Include infrastructure as code (Bicep or Terraform) configuration

Example azure.yaml structure:
```yaml
name: my-ai-app
metadata:
  template: my-ai-app@1.0.0
services:
  web:
    project: ./src/web
    language: python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/awesome-azd](https://github.com/Azure/awesome-azd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
