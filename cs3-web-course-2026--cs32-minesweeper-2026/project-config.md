---
trigger: always_on
description: These instructions guide GitHub Copilot when generating or suggesting code for this project.
---

# Copilot Instructions for Minesweeper

These instructions guide GitHub Copilot when generating or suggesting code for this project.
Follow all conventions below consistently across every file.

---

## Meaningful Naming

Always use full, descriptive names for every variable, parameter, and function. Never use
single-letter names or cryptic abbreviations.

### Variable naming

| Avoid                    | Use instead                         |
| ------------------------ | ----------------------------------- |
| `r`                      | `row`                               |
| `c`                      | `col`                               |
| `nr`                     | `neighbourRow`                      |
| `nc`                     | `neighbourCol`                      |
| `dr`                     | `directionalRow`                    |
| `dc`                     | `directionalCol`                    |
| `i`, `j` (in grid loops) | `row`, `col`                        |
| `n`                      | `count` or a domain-specific name   |
| `el`                     | `element`                           |
| `btn`                    | `button`                            |
| `val`                    | `value`                             |
| `arr`                    | domain-specific name (e.g. `cells`) |

### Function naming

- Use verb-noun pairs that clearly describe what the function does.
- Examples: `checkWinCondition`, `renderBoard`, `startTimer`.

The lab2 task specification (`docs/lab2.md`) requires these four core functions to use exactly
these names and signatures — note `neighborMines` (the cell property, no "u") vs.
`countNeighbourMines` (the function, with "u") is intentional, matching the spec verbatim:

- `generateField(rows, cols, minesCount)` — builds the board and places mines.
- `countNeighbourMines(...)` — counts mines adjacent to every cell, writing to `neighborMines`.
- `openCell(row, col)` — reveals a cell, recursing into neighbours when `neighborMines === 0`.
- `toggleFlag(row, col)` — sets/unsets a flag on a closed cell.

### General rules

- Prioritise readability over brevity.
- Names must be self-documenting: a reader should understand intent without needing a comment.
- Use camelCase for variables and functions, UPPER_SNAKE_CASE for top-level constants.

---

## Enums and Constants

Use constant objects (enum-style) instead of raw string or number literals wherever a fixed set
of values exists. Define these objects at the top of the relevant module.

### Pattern

The lab2 task specification (`docs/lab2.md`) mandates the underlying string values below for
`cell.type`, `cell.state`, and `gameState.status` — wrap them in enum-style constant objects,
but do not rename the values themselves:

```js
const CELL_TYPE = {
  EMPTY: 'empty',
  MINE: 'mine',
};

const CELL_STATE = {
  CLOSED: 'closed',
  OPENED: 'opened',
  FLAGGED: 'flagged',
};

const GAME_STATUS = {
  PROCESS: 'process',
  WIN: 'win',
  LOSE: 'lose',
};
```

### Usage

```js
// Good
cell.state = CELL_STATE.CLOSED;
gameState.status = GAME_STATUS.PROCESS;

// Bad — never use raw literals scattered through the code
cell.state = 'closed';
gameState.status = 'process';
```

### Rules

- Always reference the constant object in logic, conditionals, and assignments.
- Never duplicate the same string or magic number in more than one place.
- Group related constants into a single object.
- Place constant objects at the top of the file, before any functions.

---

## Spacing and Formatting

Consistent spacing makes the code easier to scan and review.

### Group separation

Separate distinct groups of code with a single blank line:

1. **Constants / configuration** — one block at the top.
2. **Helper / utility functions** — one block.
3. **Core logic functions** — one block.
4. **Rendering / DOM functions** — one block.
5. **`return` statement** — always preceded by a blank line when inside a function body.

### Example

```js
const CELL_STATE = {
  OPEN: 'open',
  CLOSED: 'closed',
  FLAGGED: 'flagged',
};

const DIRECTIONS = [
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, -1],
  [0, 1],
  [1, -1],
  [1, 0],
  [1, 1],
];

function countAdjacentMines(board, row, col) {
  let mineCount = 0;

  for (const [directionalRow, directionalCol] of DIRECTIONS) {
    const neighbourRow = row + directionalRow;
    const neighbourCol = col + directionalCol;

    if (isInBounds(board, neighbourRow, neighbourCol)) {
      if (board[neighbourRow][neighbourCol].hasMine) {
        mineCount++;
      }
    }
  }

  return mineCount;
}

function revealCell(board, row, col) {
  const cell = board[row][col];

  if (cell.state !== CELL_STATE.CLOSED) {
    return;
  }

  cell.state = CELL_STATE.OPEN;

  return cell;
}
```

### Rules

- One blank line between logical sections inside a function.
- Two blank lines between top-level function definitions.
- A blank line before every `return` statement (except single-expression arrow functions).
- No trailing whitespace.
- Use 2-space indentation consistently.

---

## Semantic HTML

Use the correct HTML elements for their intended purpose. Never use generic `<div>` or `<span>`
elements for interactive controls or landmark regions.

### Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cs3-web-course-2026/cs32-minesweeper-2026](https://github.com/cs3-web-course-2026/cs32-minesweeper-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
