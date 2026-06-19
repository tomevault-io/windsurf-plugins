---
trigger: always_on
description: >
---


# PhiaUI Design System

Linguagem visual obrigatória para todos os componentes PhiaUI.
Para padrões de **código** (API, cn/1, variants, testes, segurança) → ver `phia-component/SKILL.md`.

---

## 1. Filosofia & Referência

- **shadcn/ui** é a referência primária — consultar via Context7 (`/shadcn-ui/ui`) antes de implementar
- **Tailwind CSS v4** — CSS-first config via `@theme` / `@source` / `@utility` (sem `tailwind.config.js`)
- **OKLCH** — paleta de cores em espaço perceptualmente uniforme (ampla gama, dark mode preciso)
- **Zero libs externas de UI** — sem Heroicons npm, sem Alpine, sem class-variance-authority
- **Semantic tokens sempre** — nunca usar cores Tailwind brutas (`blue-500`) em componentes

---

## 2. Sistema de Cores

### Tokens Semânticos (todos mapeados em `priv/static/theme.css`)

| Token | Light | Dark | Uso |
|-------|-------|------|-----|
| `background` | oklch(1 0 0) | oklch(0.145 0 0) | Fundo da página |
| `foreground` | oklch(0.145 0 0) | oklch(0.985 0 0) | Texto principal |
| `primary` | oklch(0.205 0 0) | oklch(0.922 0 0) | CTA, botão principal |
| `primary-foreground` | oklch(0.985 0 0) | oklch(0.205 0 0) | Texto sobre primary |
| `secondary` | oklch(0.97 0 0) | oklch(0.269 0 0) | Ações secundárias |
| `muted` | oklch(0.97 0 0) | oklch(0.269 0 0) | Áreas neutras |
| `muted-foreground` | oklch(0.556 0 0) | oklch(0.708 0 0) | Texto de suporte |
| `accent` | oklch(0.97 0 0) | oklch(0.269 0 0) | Hover em menus |
| `destructive` | oklch(0.577 0.245 27.325) | oklch(0.704 0.191 22.216) | Erros, perigo |
| `border` | oklch(0.922 0 0) | oklch(0.269 0 0) | Bordas e divisores |
| `input` | oklch(0.922 0 0) | oklch(0.269 0 0) | Bordas de inputs |
| `ring` | oklch(0.708 0 0) | oklch(0.556 0 0) | Focus ring |
| `card` / `card-foreground` | background / foreground | idem | Superfície de card |
| `popover` / `popover-foreground` | background / foreground | idem | Overlays flutuantes |
| `sidebar` / `sidebar-*` | — | — | Navegação lateral |
| `chart-1` … `chart-5` | oklch variados | oklch variados | Visualizações |

### Presets de Cor (8 temas em `priv/static/themes/`)

`zinc` (padrão), `slate`, `blue`, `rose`, `orange`, `green`, `violet`, `neutral`

Cada preset redefine apenas os tokens `primary` / `primary-foreground` — demais tokens são neutros.

### Modificadores de Opacidade

```heex
<%# hover: reduz opacidade em vez de cor diferente %>
class="bg-primary hover:bg-primary/90"
class="bg-muted hover:bg-muted/80"
class="text-muted-foreground/70"
```

Usar `/80`, `/90`, `/70` para estados de hover/active — **nunca** inventar nova cor.

### Regra de Ouro

```
✅ bg-primary       ✅ bg-destructive     ✅ bg-muted
❌ bg-blue-500      ❌ bg-red-600         ❌ bg-gray-100
```

---

## 3. Tipografia

### Famílias

```css
/* priv/static/theme.css */
--font-sans: ui-sans-serif, system-ui, sans-serif   /* padrão absoluto */
--font-mono: ui-monospace, 'Cascadia Code', monospace /* código, kbd, OTP */
```

Nenhum componente deve importar Google Fonts ou fontes externas.

### Escala de Tamanho (8 níveis)

| Classe | rem | px | Uso típico |
|--------|-----|----|------------|
| `text-xs` | 0.75 | 12 | Badges, labels de campo, metadados |
| `text-sm` | 0.875 | 14 | Corpo padrão, itens de lista, inputs |
| `text-base` | 1 | 16 | Parágrafos, descrições longas |
| `text-lg` | 1.125 | 18 | Subtítulos de seção |
| `text-xl` | 1.25 | 20 | Títulos de card, dialog headings |
| `text-2xl` | 1.5 | 24 | Page headings |
| `text-3xl` | 1.875 | 30 | Hero headings |
| `text-4xl` | 2.25 | 36 | Display / landing titles |

### Pesos

| Classe | Uso |
|--------|-----|
| `font-normal` | Corpo de texto, descrições |
| `font-medium` | Labels, botões, itens de menu |
| `font-semibold` | Títulos de componente, card_title |
| `font-mono` | Código, kbd, OTP inputs, timestamps técnicos |

### Line Height

- `leading-none` (1) — headings display
- `leading-tight` (1.25) — títulos compactos
- `leading-normal` (1.5) — padrão de corpo
- `leading-relaxed` (1.625) — texto longo / prose

---

## 4. Sistema de Espaçamento

Base: **4px** (1 unidade Tailwind = 4px). Todo espaçamento é múltiplo de 4.

### Padding por Tipo de Componente

| Componente | Padrão |
|------------|--------|
| Botões sm | `px-3 py-1.5` (12px / 6px) |
| Botões default | `px-4 py-2` (16px / 8px) |
| Botões lg | `px-8 py-2.5` (32px / 10px) |
| Inputs | `px-3 py-2` (12px / 8px) |
| Cards | `p-6` (24px) |
| Card header/footer | `px-6 py-4` (24px / 16px) |
| Popover / Dropdown | `p-1` (4px) para container, `px-2 py-1.5` para item |
| Dialog | `p-6` corpo, `p-6 pt-0` footer |
| Alert | `p-4` (16px) |
| Badge | `px-2.5 py-0.5` (10px / 2px) |
| Kbd | `px-1.5 py-0.5` (6px / 2px) |
| Tooltip | `px-3 py-1.5` (12px / 6px) |

### Gap Patterns

```heex
<%# Stack vertical padrão %>
class="flex flex-col gap-4"
<%# Inline group de botões %>
class="flex items-center gap-2"
<%# Grid de cards %>
class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4"
<%# Formulário %>
class="space-y-4"
```

### Margin

- Nunca usar `m-*` em componentes de biblioteca — responsabilidade do consumer
- Exceção: `mt-1.5` para mensagens de erro abaixo de inputs

---

## 5. Raio de Borda

Escala baseada em `--radius: 0.625rem` (10px) definida no tema:

| Classe | Valor | Uso |
|--------|-------|-----|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [charlenopires/PhiaUI](https://github.com/charlenopires/PhiaUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
