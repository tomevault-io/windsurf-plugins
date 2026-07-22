---
trigger: always_on
description: This is a **multi-app collection** of 41 minimal "Hello World" demos across JavaScript/TypeScript frameworks and libraries. Each subfolder is an independent project with its own `package.json` (or equivalent).
---

# AI-Ready Repo — Agent Guide

This is a **multi-app collection** of 41 minimal "Hello World" demos across JavaScript/TypeScript frameworks and libraries. Each subfolder is an independent project with its own `package.json` (or equivalent).

## Repository Structure

```
hello-worlds/
├── .github/
│   ├── copilot-instructions.md          # AI coding conventions for this repo
│   ├── dependabot.yml                   # GitHub Actions dependency updates
│   ├── workflows/
│   │   ├── ci.yml                       # PR validation (checks new projects build)
│   │   └── copilot-setup-steps.yml      # Cloud agent setup
│   ├── ISSUE_TEMPLATE/
│   │   ├── new-hello-world-project.md   # Template for proposing new frameworks
│   │   └── bug-report.md               # Bug report template
│   └── PULL_REQUEST_TEMPLATE.md         # PR checklist
├── alpine/                              # Alpine.js hello world
├── angular/                             # Angular hello world
├── angular-universal/                   # Angular Universal hello world
├── angularjs/                           # AngularJS hello world
├── aurelia/                             # Aurelia hello world
├── babylonjs/                           # Babylon.js hello world
├── backbone/                            # Backbone.js hello world
├── blazor/                              # Blazor hello world (.NET)
├── eleventy/                            # Eleventy hello world
├── elm/                                 # Elm hello world
├── ember/                               # Ember.js hello world
├── flutter/                             # Flutter hello world (Dart)
├── gatsby/                              # Gatsby hello world
├── glimmer/                             # Glimmer.js hello world
├── gridsome/                            # Gridsome hello world
├── html/                                # Plain HTML hello world
├── hugo/                                # Hugo hello world (Go)
├── hyperapp/                            # Hyperapp hello world
├── ionic-angular/                       # Ionic Angular hello world
├── ionic-react/                         # Ionic React hello world
├── javascript/                          # Plain JavaScript hello world
├── jquery/                              # jQuery hello world
├── knockoutjs/                          # Knockout.js hello world
├── lit-element/                         # LitElement hello world
├── marko/                               # Marko hello world
├── meteor/                              # Meteor hello world
├── mithril/                             # Mithril hello world
├── nextjs/                              # Next.js hello world
├── nuxtjs/                              # Nuxt.js hello world
├── polymer/                             # Polymer hello world
├── preact/                              # Preact hello world
├── react/                               # React hello world
├── riot/                                # Riot.js hello world
├── scully/                              # Scully hello world
├── stencil/                             # Stencil hello world
├── svelte/                              # Svelte hello world
├── threejs/                             # Three.js hello world
├── typescript/                          # TypeScript hello world
├── vue/                                 # Vue.js hello world
├── vuepress/                            # VuePress hello world
├── web-component/                       # Web Component hello world
├── AGENTS.md                            # This file
├── CHANGELOG.md                         # Version history
├── CODE_OF_CONDUCT.md                   # Community guidelines
├── CONTRIBUTING.md                      # Contribution guide
├── LICENSE                              # MIT
└── README.md                            # Project overview with build matrix
```

## Tech Stack

- **Languages:** JavaScript, TypeScript, HTML, CSS
- **Non-JS projects:** Blazor (.NET/C#), Flutter (Dart), Hugo (Go), Elm
- **Build tools:** Varies per project — webpack, rollup, Angular CLI, Create React App, Gatsby CLI, etc.
- **Package manager:** npm (each project has its own `package.json`)
- **No centralized build or test system** — each subfolder is independent

## Build & Run

There is no root-level build. Each project builds independently:

```bash
cd <project-folder>
npm install
npm start        # or npm run dev
```

To build for production:

```bash
cd <project-folder>
npm run build    # output folder varies per framework
```

Refer to the build matrix in `README.md` for each project's build command and output folder.

### Exceptions

- **Blazor**: Requires .NET SDK → `dotnet build`
- **Flutter**: Requires Flutter SDK → `flutter build web`
- **Hugo**: Requires Hugo binary (installed via npm)
- **Static projects** (alpine, angularjs, babylonjs, backbone, html, hyperapp, javascript, jquery, threejs, web-component): No build step — serve directly

## Testing

There is no centralized test suite. Individual projects may have tests, but the repo convention is to **remove all tests** and keep demos minimal.

To verify a project works:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnpapa/hello-worlds](https://github.com/johnpapa/hello-worlds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
