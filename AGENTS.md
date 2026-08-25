# AGENTS.md

## Run the game
Open `src/index.html` directly in a browser. No build step, no package manager.

## Architecture
- **Vanilla JS/HTML/CSS** — no frameworks, no modules, no bundler.
- Scripts are loaded in strict order in `src/index.html`:
  `maze.js` → `game.js` → `render.js` → `main.js`
- All inter-file communication happens via globals attached to `window`.

## Key source files
| File | Role |
|---|---|
| `src/js/maze.js` | Maze grid (28×31), parsed from string matrix. Exports `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS` to `window`. |
| `src/js/game.js` | Game state & rules. `createGame()` copies `MAZE` into a mutable `grid`. `update()` runs the frame logic. |
| `src/js/render.js` | Canvas drawing at 20px/tile (560×620 canvas). `draw()` is the single entry point. |
| `src/js/main.js` | Game loop (`requestAnimationFrame`), keyboard input, overlay UI. |

## Grid encoding
1 = wall, 2 = dot, 3 = ghost-pen door (blocks Pac-Man only), 0 = empty

## OpenCode spec workflow
This repo uses spec-driven development via two OpenCode skills:
- `spec` — design features before coding
- `spec-impl` — implement approved specs (creates a git branch automatically)
