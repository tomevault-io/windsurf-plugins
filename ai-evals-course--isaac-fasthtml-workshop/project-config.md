---
trigger: always_on
description: The main file for the project is [main_sqlite.py](mdc:fasthtml_app/main_sqlite.py) It uses MonsterUI for styling.  MonsterUI is a python first UI component library that primarily leverages FrankenUI and Tailwind, but also includes headers and functionality form DaisyUI, Katex, HighlightJS, and others.
---


# Project Structure

The main file for the project is [main_sqlite.py](mdc:fasthtml_app/main_sqlite.py) It uses MonsterUI for styling.  MonsterUI is a python first UI component library that primarily leverages FrankenUI and Tailwind, but also includes headers and functionality form DaisyUI, Katex, HighlightJS, and others.

You can run the server with the command `python app/main_sqlite.py`, which will start the server on http://localhost:5001.

# Tech Stack

- FastHTML is the web application framework.  It is built on top of starlette and uvicorn.
- MonsterUI is a UI component library designed to work well with FastHTML
- Fastlite is a sqlite library that is a small wrapper on top of sqlite-utils

# Key documentation files:

These are relevant documents that should be referenced while building a FastHTML App.

- [fasthtml.mdc](mdc:.cursor/rules/fasthtml.mdc): Minimal HTMX integration exaples to show how HTMX can be used with fasthtml
- [db.md](mdc:ref_docs/db.md): MiniDataAPI Spec for database operations
- [monsterui_api.md](mdc:ref_docs/monsterui_api.md).md : MonsterUI full API list and idiomatic examples for UI components

# FastHTML examples

Reference these examples when constructing new FastHTML applications.

- [annotation.md](mdc:ref_docs/annotation.md): A siple example of a annotation app to evaluate search results.  
- Adv_app: Example FastHTML To-Do-List application that demonstrates core FastHTML features including authentication, HTMX integration, and database operations. It allows users to create, edit, delete, and reorder todos with markdown support, using SQLite for storage.

# FastHTML Rules

- Use `serve()` directly - no need for uvicorn or separate ASGI server
- Not compatible with FastAPI syntax - FastHTML is for HTML-first apps, not API services
- Define routes with decorators and return HTML components or strings
- Use python FastTags (ie `Div`, `P`) instead of raw HTML where possible
- Use HTMX for interactive features, vanilla JS where needed. No React/Vue/Svelte

# UI Design Elements with MonsterUI

- Use defaults as much as possible, for example `Container` in monsterui already has defaults for margins
- Use `*T` for button styling consistency, for example `ButtonT.destructive` for a red delete button or `ButtonT.primary` for a CTA button
- Use `Label*` functions for forms as much as possible (e.g. `LabelInput`, `LabelRange`) which creates and links both the `FormLabel` and user input appropriately to avoid boiler plate

## Basic Complete App Example

```python
from fasthtml.common import *
from monsterui.all import *

app, rt = fast_app(hdrs=Theme.blue.headers()) # Use MonsterUI blue theme

@rt
def index():
    socials = (('github','https://github.com/AnswerDotAI/MonsterUI'),
               ('twitter','https://twitter.com/isaac_flath/'),
               ('linkedin','https://www.linkedin.com/in/isaacflath/'))
    return Titled("Your First App",
        Card(
            P("Your first MonsterUI app", cls=TextPresets.muted_sm),
            # LabelInput, DivLAigned, and UkIconLink are non-semantic MonsterUI FT Components,
            LabelInput('Email', type='email', required=True),
            footer=DivLAligned(*[UkIconLink(icon,href=url) for icon,url in socials])))
```

## Card and Flex Layout Components Example

```python
def TeamCard(name, role, location="Remote"):
    icons = ("mail", "linkedin", "github")
    return Card(
        DivLAligned(
            DiceBearAvatar(name, h=24, w=24),
            Div(H3(name), P(role))),
        footer=DivFullySpaced(
            DivHStacked(UkIcon("map-pin", height=16), P(location)),
            DivHStacked(*(UkIconLink(icon, height=16) for icon in icons))))
```

## Forms and User Inputs Example

```python
def MonsterForm():
    relationship = ["Parent",'Sibling', "Friend"]
    return Div(
        DivCentered(
            H3("Emergency Contact Form"),
            P("Please fill out the form completely", cls=TextPresets.muted_sm)),
        Form(
            Grid(LabelInput("Name",id='name'),LabelInput("Email",     id='email')),
            H3("Relationship to patient"),
            Grid(*[LabelCheckboxX(o) for o in relationship], cols=4, cls='space-y-3'),
            DivCentered(Button("Submit Form", cls=ButtonT.primary))),
        cls='space-y-4')
```

## Markdown Text Styling Example

```python
render_md("""
# My Document

> Important note here

+ List item with **bold**
+ Another with `code`

```python
def hello():
    print("world")
```
""")
```

## Semantic Text Styling Example

```python
def SemanticText():
    return Card(
        H1("MonsterUI's Semantic Text"),
        P(
            Strong("MonsterUI"), " brings the power of semantic HTML to life with ",
            Em("beautiful styling"), " and ", Mark("zero configuration"), "."),
        Blockquote(
            P("Write semantic HTML in pure Python, get modern styling for free."),
            Cite("MonsterUI Team")),
        footer=Small("Released February 2025"),)
```

# Data Storage

- `fastlite` (SQLite) included and preferred.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-evals-course/isaac-fasthtml-workshop](https://github.com/ai-evals-course/isaac-fasthtml-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
