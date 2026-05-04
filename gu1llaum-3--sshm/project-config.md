---
trigger: always_on
description: These project-level instructions tell Copilot how to generate idiomatic, production-quality Go code using the Bubble Tea ecosystem. **Follow and prefer these rules over generic patterns.**
---

# GitHub Copilot Instructions for Go + Bubble Tea (TUI)

These project-level instructions tell Copilot how to generate idiomatic, production-quality Go code using the Bubble Tea ecosystem. **Follow and prefer these rules over generic patterns.**

---

## 1) Project Scope & Goals

* Build terminal UIs with **[Bubble Tea](https://github.com/charmbracelet/bubbletea)** and **Bubbles** components.
* Use **Lip Gloss** for styling and **Huh**/**Bubbles forms** for prompts where useful.
* Favor **small, composable models** and **message-driven state**.
* Prioritize **maintainability, testability, and clear separation** of update vs. view.

---

## 2) Go Conventions to Prefer

* Go version: **1.22+**.
* Module: `go.mod` with minimal, pinned dependencies; use `go get -u` only deliberately.
* Code style: `gofmt`, `go vet`, `staticcheck` (when available), `golangci-lint`.
* Names: short, meaningful; exported symbols require GoDoc comments.
* Errors: return wrapped errors with `%w` and `errors.Is/As`. No panics for flow control.
* Concurrency: use `context.Context` and `errgroup` where applicable. Avoid goroutine leaks; cancel contexts in `Quit`/`Stop`.
* Testing: `*_test.go`, table-driven tests, golden tests for `View()` when helpful.
* Logging: prefer structured logs (e.g., `slog`) and keep logs separate from UI rendering.

---

## 3) Bubble Tea Architecture Rules

### 3.1 Model layout

```go
// Model holds all state needed to render and update.
type Model struct {
    width, height int
    ready        bool

    // Domain state
    items   []Item
    cursor  int
    err     error

    // Child components
    list    list.Model
    spinner spinner.Model

    // Styles
    styles  Styles
}
```

**Guidelines**

* Keep **domain state** (data) separate from **UI components** (Bubbles models) and **styles**.
* Add a `Styles` struct to centralize Lip Gloss styles; initialize once.
* Track terminal size (`width`, `height`); re-calc layout on `tea.WindowSizeMsg`.

### 3.2 Init

* Return **batch** of startup commands for IO (e.g., loading data) and component inits.
* Never block in `Init`; do IO in `tea.Cmd`s.

```go
func (m Model) Init() tea.Cmd {
    return tea.Batch(m.spinner.Tick, loadItemsCmd())
}
```

### 3.3 Update

* **Pure function** style: transform `Model` + `Msg` → `(Model, Cmd)`.
* Always handle `tea.WindowSizeMsg` to set `m.width`/`m.height` and recompute layout.
* Use **type-switched** message handling; push side effects into `tea.Cmd`s.
* Bubble components: call `Update(msg)` on children and **return their Cmd**.

```go
func (m Model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.WindowSizeMsg:
        m.width, m.height = msg.Width, msg.Height
        m.styles = NewStyles(m.width) // recompute if responsive
        return m, nil

    case errMsg:
        m.err = msg
        return m, nil

    case itemsLoaded:
        m.items = msg
        return m, nil
    }

    // delegate to children last
    var cmd tea.Cmd
    m.spinner, cmd = m.spinner.Update(msg)
    return m, cmd
}
```

### 3.4 View

* **Never** mutate state in `View()`.
* Compose layout with Lip Gloss; gracefully handle small terminals.
* Put errors and help at the bottom.

```go
func (m Model) View() string {
    if !m.ready {
        return m.styles.Loading.Render(m.spinner.View() + " Loading…")
    }
    main := lipgloss.JoinVertical(lipgloss.Left,
        m.styles.Title.Render("My App"),
        m.list.View(),
    )
    if m.err != nil {
        main += "\n" + m.styles.Error.Render(m.err.Error())
    }
    return m.styles.App.Render(main)
}
```

### 3.5 Messages & Commands

* Define **typed messages** for domain events, not raw strings.
* Each async operation returns a **message type**; handle errors with a dedicated `errMsg`.

```go
type itemsLoaded []Item

type errMsg error

func loadItemsCmd() tea.Cmd {
    return func() tea.Msg {
        items, err := fetchItems()
        if err != nil { return errMsg(err) }
        return itemsLoaded(items)
    }
}
```

### 3.6 Keys & Help

* Centralize keybindings and help text. Prefer `bubbles/key` + `bubbles/help`.

```go
type keyMap struct {
    Up, Down, Select, Quit key.Binding
}

var keys = keyMap{
    Up:     key.NewBinding(key.WithKeys("up", "k"), key.WithHelp("↑/k", "up")),
    Down:   key.NewBinding(key.WithKeys("down", "j"), key.WithHelp("↓/j", "down")),
    Select: key.NewBinding(key.WithKeys("enter"), key.WithHelp("enter", "select")),
    Quit:   key.NewBinding(key.WithKeys("q", "ctrl+c"), key.WithHelp("q", "quit")),
}
```

Handle keys in `Update` using `key.Matches(msg, keys.X)` and show a `help.Model` in the footer.

### 3.7 Submodels (Component Composition)

* For complex screens, create **submodels** with their own `(Model, Init, Update, View)` and wire them into a parent.
* Exchange messages via **custom Msg types** and/or **parent state**.
* Keep submodels **pure**; IO still goes through parent-level `tea.Cmd`s or via submodel commands returned to parent.

### 3.8 Program Options


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gu1llaum-3/sshm](https://github.com/Gu1llaum-3/sshm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
