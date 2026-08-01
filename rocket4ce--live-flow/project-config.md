---
trigger: always_on
description: This is a web application written using the Phoenix web framework.
---

This is a web application written using the Phoenix web framework.

## Skills Disponibles

Este proyecto tiene skills especializadas para ayudar en tareas especificas. **Usa estas skills cuando trabajes en areas relacionadas.**

### Como Usar Skills

Invoca una skill con `/skill-name` cuando necesites conocimiento especializado:

| Skill | Comando | Usar Cuando |
|-------|---------|-------------|
| **LiveFlow Debug** | `/liveflow-debug` | Hay errores de eventos, conexiones o rendering en el flow |
| **LiveFlow Patterns** | `/liveflow-patterns` | Implementas nuevas funcionalidades o disenias arquitectura del flow |
| **XYFlow Pan/Zoom** | `/xyflow-panzoom` | Implementas navegacion del viewport, zoom, o transformaciones de coordenadas |
| **XYFlow Drag** | `/xyflow-drag` | Implementas arrastre de nodos, seleccion multiple, snap-to-grid |
| **XYFlow Edges** | `/xyflow-edges` | Implementas conectores entre nodos, paths SVG (bezier/step/smoothstep) |
| **XYFlow Connections** | `/xyflow-connections` | Implementas handles, validacion de conexiones, crear nuevas edges |
| **Skill Authoring** | `/skill-authoring` | Necesitas crear o corregir una skill |

### Cuando Invocar Skills Automaticamente

1. **Error en LiveFlow** → Usa `/liveflow-debug`
2. **Nuevo feature de flow** → Usa `/liveflow-patterns` + skill especifica (drag/edges/etc)
3. **Calculos de paths SVG** → Usa `/xyflow-edges`
4. **Sistema de conexiones** → Usa `/xyflow-connections`
5. **Crear nueva skill** → Usa `/skill-authoring`

### Archivos de Skills

```
.claude/skills/
├── liveflow-debug/SKILL.md
├── liveflow-patterns/SKILL.md
├── skill-authoring/SKILL.md
├── xyflow-connections/SKILL.md
├── xyflow-drag/SKILL.md
├── xyflow-edges/SKILL.md
└── xyflow-panzoom/SKILL.md
```

---

## Project guidelines

- Use `mix precommit` alias when you are done with all changes and fix any pending issues
- Use the already included and available `:req` (`Req`) library for HTTP requests, **avoid** `:httpoison`, `:tesla`, and `:httpc`. Req is included by default and is the preferred HTTP client for Phoenix apps

### Phoenix v1.8 guidelines

- **Always** begin your LiveView templates with `<Layouts.app flash={@flash} ...>` which wraps all inner content
- The `MyAppWeb.Layouts` module is aliased in the `my_app_web.ex` file, so you can use it without needing to alias it again
- Anytime you run into errors with no `current_scope` assign:
  - You failed to follow the Authenticated Routes guidelines, or you failed to pass `current_scope` to `<Layouts.app>`
  - **Always** fix the `current_scope` error by moving your routes to the proper `live_session` and ensure you pass `current_scope` as needed
- Phoenix v1.8 moved the `<.flash_group>` component to the `Layouts` module. You are **forbidden** from calling `<.flash_group>` outside of the `layouts.ex` module
- Out of the box, `core_components.ex` imports an `<.icon name="hero-x-mark" class="w-5 h-5"/>` component for for hero icons. **Always** use the `<.icon>` component for icons, **never** use `Heroicons` modules or similar
- **Always** use the imported `<.input>` component for form inputs from `core_components.ex` when available. `<.input>` is imported and using it will save steps and prevent errors
- If you override the default input classes (`<.input class="myclass px-2 py-1 rounded-lg">)`) class with your own values, no default classes are inherited, so your
custom classes must fully style the input

### JS and CSS guidelines

- **Use Tailwind CSS classes and custom CSS rules** to create polished, responsive, and visually stunning interfaces.
- Tailwindcss v4 **no longer needs a tailwind.config.js** and uses a new import syntax in `app.css`:

      @import "tailwindcss" source(none);
      @source "../css";
      @source "../js";
      @source "../../lib/my_app_web";

- **Always use and maintain this import syntax** in the app.css file for projects generated with `phx.new`
- **Never** use `@apply` when writing raw css
- **Always** manually write your own tailwind-based components instead of using daisyUI for a unique, world-class design
- Out of the box **only the app.js and app.css bundles are supported**
  - You cannot reference an external vendor'd script `src` or link `href` in the layouts
  - You must import the vendor deps into app.js and app.css to use them
  - **Never write inline <script>custom js</script> tags within templates**

### UI/UX & design guidelines

- **Produce world-class UI designs** with a focus on usability, aesthetics, and modern design principles
- Implement **subtle micro-interactions** (e.g., button hover effects, and smooth transitions)
- Ensure **clean typography, spacing, and layout balance** for a refined, premium look
- Focus on **delightful details** like hover effects, loading states, and smooth page transitions


<!-- usage-rules-start -->

<!-- phoenix:elixir-start -->
## Elixir guidelines

- Elixir lists **do not support index based access via the access syntax**

  **Never do this (invalid)**:

      i = 0
      mylist = ["blue", "green"]
      mylist[i]

  Instead, **always** use `Enum.at`, pattern matching, or `List` for index based list access, ie:

      i = 0
      mylist = ["blue", "green"]
      Enum.at(mylist, i)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rocket4ce/live_flow](https://github.com/rocket4ce/live_flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
