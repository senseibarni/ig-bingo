---
name: Bingo canvas grid
overview: A single static HTML file plus bingo_ig.png — a 1200×1200 canvas that CSS-scales, uses the PNG as the background, and lets users toggle 16 squares with localStorage, a reset button, and a hook for future winning combinations.
todos:
  - id: copy-png
    content: Copy bingo_ig.png into the project root (from the uploaded asset)
    status: completed
  - id: create-html
    content: Create index.html with canvas, CSS scale, PNG background, hit-testing, localStorage, reset, combination stubs, and createBoard() factory
    status: completed
isProject: false
---

# Bingo canvas (MVP)

Two static files, no bundler (simpler than Parcel, ready to drop on any static server):

- [index.html](index.html) — markup, CSS, and JS in one file
- [bingo_ig.png](bingo_ig.png) — copied from the uploaded card art

## Layout math (logical 1200×1200)

- Top 112, left 108, right 116, gutter 8
- Square size **238×238**; bottom leftover is 112
- Cell `(c, r)`: `x = 108 + c × 246`, `y = 112 + r × 246` (246 = 238 + 8)
- Index `i = r * 4 + c` (0–15)

## Scaling

Draw only in a **fixed 1200×1200 bitmap**. CSS scales the canvas (and therefore the PNG and overlays together):

- Page: black background, column flex, canvas + Reset centered
- Canvas: `width: min(1200px, 100vw - 16px, calc(100dvh - 80px))`, `height: auto`, `aspect-ratio: 1`
- Hits map CSS pixels → logical coords via `getBoundingClientRect()` and `1200 / rect.width`

## Drawing order

1. Draw `bingo_ig.png` full-bleed (`drawImage` 0,0,1200,1200)
2. For each selected cell: cyan→magenta linear gradient fill at **50% opacity**, then a matching gradient (or cyan/magenta) **border** at full opacity
3. If any combination matches (later): apply that combination’s fill/border on the relevant cells (or whole board), on top of the selected overlay

Do not stroke empty cells — the PNG already has the grid.

## Interaction

- `pointerdown` on the canvas: hit-test the 16 rects, **toggle** that cell, save, redraw
- Ignore clicks in gutters/margins
- **Reset** is an HTML button below the canvas. On click, show a confirm dialog first (`window.confirm`, Polish copy to match the card, e.g. "Czy na pewno chcesz zresetować kartę?"). Only if the user confirms: clear all 16 flags, save, redraw. Cancel leaves the board unchanged.

## Persistence

`localStorage` key e.g. `bingo-ig-selected`: JSON array of 16 booleans. Load on init; missing/invalid data → all false.

## JS structure (MVP + later)

Keep the board as a factory so a second canvas can be added later without a rewrite:

```js
function createBoard(options) {
  // options: canvas, storageKey, combinations
  // returns { reset(), getSelected(), redraw() }
}
```

MVP calls it once. A later duplicate uses a second canvas + a different `storageKey`.

**Combinations** (empty for MVP, filled in later — 3 or 4 rules):

```js
const COMBINATIONS = [
  // { id, cells: [0, 1, 2, 3], fill, border }
];
```

After every toggle/reset, `matchCombinations(selected)` returns the matching rule(s). Drawing reads that result so swapping in real combos is data-only, not a new event model.

## Deploy

Upload `index.html` and `bingo_ig.png` to the same directory. Open in a browser (or any static host). No install, no build.
