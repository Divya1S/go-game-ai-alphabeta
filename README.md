# Stone Garden — Go AI

> A production-grade, single-file Go board game with a Minimax α-β AI opponent.

**Live Demo → [https://go-game-ai.netlify.app/](https://go-game-ai.netlify.app/)**

![Stone Garden](https://img.shields.io/badge/version-4.0-C8923A?style=flat-square)
![No Dependencies](https://img.shields.io/badge/dependencies-zero-4CAF80?style=flat-square)
![Single File](https://img.shields.io/badge/build-single--file-9B7EDE?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)

---

## Overview

Stone Garden is a complete, dependency-free implementation of the ancient board game Go — playable in any modern browser straight from a single HTML file. It features a hand-crafted AI opponent powered by Minimax search with alpha-beta pruning, a Zobrist hashing super-ko rule, Japanese scoring, and a full game management suite including undo/redo, SGF import/export, and a frame-by-frame review mode.

The entire application — engine, AI, renderer, and UI — is contained in one self-hosted `stone-garden-go.html` file with no external dependencies, no build step, and no server required.

---

## Features

### Gameplay
- **Three board sizes** — 5×5, 7×7, and 9×9
- **Japanese scoring** with configurable komi (0.5 – 20, including presets)
- **Pass and resign** — full rule compliance
- **Legal move enforcement** — occupied intersections, suicide, and super-ko all rejected
- **Two consecutive passes** end the game automatically

### AI Engine
- **Minimax with alpha-beta pruning** — exact game-tree search with guaranteed optimal play at configured depth
- **Transposition table** — 400k-entry Zobrist hash cache eliminates redundant subtrees
- **Super-ko detection** — full position history tracked via 64-bit Zobrist hashes (two 32-bit halves XOR'd) to prevent repeated board positions
- **Move ordering** — captures, atari threats, and center proximity scored and sorted before search, maximising pruning efficiency
- **Static evaluation** — multi-factor heuristic covering territory (flood-fill), captures, liberty pressure, atari detection, stone count, and center control
- **Three difficulty levels** — Beginner (random), Intermediate (3 ply), Strong (up to 6 ply on 5×5)
- **Real-time performance stats** — nodes/second and search depth displayed live

### UI & Quality of Life
- **Hint system** — AI suggests the best move with an animated pulsing ring `[H]`
- **Undo / Redo** — full snapshot stack; undo reverts both the AI reply and your move `[Z] / [Y]`
- **Review mode** — replay any finished or in-progress game frame by frame `[↵]`
- **Territory overlay** — visual flood-fill territory map with legend `[T]`
- **Move number overlay** — display sequence numbers on all stones `[N]`
- **SGF import/export** — FF[4]-compliant Smart Game Format, copy to clipboard or paste to load `[S]`
- **Capture particles** — physics-based burst animation on stone captures
- **Stone placement animation** — satisfying scale-bounce on every move
- **Hover ghost** — semi-transparent preview stone follows the cursor before placement
- **Last move marker** — small dot indicates the most recently played stone

### Design
- Dark premium aesthetic — deep ink-black background, amber wood board, warm typography
- Canvas 2D renderer — radial-gradient stones with specular highlights and drop shadows
- Responsive layout — full three-panel view on desktop, gracefully collapses on tablet and mobile
- Touch support — `touchend` handler for mobile browsers
- Full keyboard shortcut suite (see below)

---

## Quick Start

No installation. No build step. No server.

```bash
# Option 1 — open directly in a browser
open stone-garden-go.html

# Option 2 — serve locally (avoids any file:// quirks)
python3 -m http.server 8080
# then visit http://localhost:8080/stone-garden-go.html

# Option 3 — play online
# https://go-game-ai.netlify.app/
```

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `H` | Request hint |
| `P` | Pass your turn |
| `Q` | Resign |
| `R` | New game |
| `Z` | Undo |
| `Y` | Redo |
| `T` | Toggle territory overlay |
| `N` | Toggle move numbers |
| `S` | Open SGF panel |
| `,` | Open settings |
| `↵` | Toggle review mode |
| `← →` | Undo / step forward in review |
| `Home / End` | Jump to first / last move in review |
| `Esc` | Close modal / exit review |

---

## Architecture

The application is structured in 21 clearly separated sections within a single file, written to the standard of a production codebase.

```
stone-garden-go.html
│
├── SECTION 1  · Constants
│     CELL_EMPTY / CELL_BLACK / CELL_WHITE
│     SGF_ALPHA, BOARD_ALPHA coordinate alphabets
│
├── SECTION 2  · Zobrist Hash Table
│     64-bit hashes via two XOR'd Uint32 values
│     Pre-computed for all 81 cells × 2 colours
│
├── SECTION 3  · GoEngine (class)
│     Board state, move legality (tryMove)
│     Flood-fill group analysis (_group)
│     Capture resolution, super-ko enforcement
│     Pass, resign, undo snapshot system
│     Japanese territory scoring (getTerritoryMap, getScore)
│
├── SECTION 4  · Transposition Table
│     Hash map with 400k entry eviction
│     Exact / lower-bound / upper-bound flags
│
├── SECTION 5  · Static Evaluation
│     evalBoard() — territory + captures + liberty pressure
│     + atari detection + stone count + center control
│
├── SECTION 6  · Move Ordering
│     orderedMoves() — scored + sorted candidate generator
│     Beam-search width: 18 / 15 / 13 by board size
│
├── SECTION 7  · Minimax α-β Search
│     minimax() — recursive with TT lookup/store
│     computeAIMove() — root search for White
│     getSearchDepth() — depth mapping per difficulty + size
│
├── SECTION 8  · Application State (App object)
│     All mutable state in one place; no implicit globals
│
├── SECTION 9  · Canvas Initialisation
│     DPR-aware sizing, geometry constants
│
├── SECTION 10 · Coordinate Helpers
│     gridToXY, xyToGrid, coordStr
│
├── SECTION 11 · Renderer
│     drawBoard()  — wood gradient, grain, grid, stars, labels
│     drawStone()  — radial gradient, shadow, specular
│     drawStones() — full board pass with optional move numbers
│     drawTerritory, drawHover, drawHint, drawParticles
│     render()     — scene compositor + conditional RAF loop
│
├── SECTION 12 · Animation Helpers
│     animPlace()    — scale-bounce stone placement
│     spawnCaptures() — physics particle burst
│
├── SECTION 13 · Review Mode
│     buildReviewSnapshots(), applyReviewSnapshot()
│     enterReview / exitReview / reviewStep / reviewGoTo
│
├── SECTION 14 · Undo / Redo
│     captureFullState / applyFullState
│     doUndo — reverts AI + player move pair
│     doRedo — restores from snapshot stack
│
├── SECTION 15 · Game Flow
│     execHumanMove → animPlace → spawnCaptures → afterPlayerMove
│     scheduleAI → computeAIMove (setTimeout) → animPlace → afterAI
│     doPass, doResign, endGame, requestHint
│
├── SECTION 16 · New Game
│     Full state reset, engine reinit, canvas reinit
│
├── SECTION 17 · UI Updates
│     updateUI, updateHistoryList, setStatusDot
│     toggleTerr, toggleNums
│
├── SECTION 18 · Settings & Modals
│     selectOpt, setPending, openSettings, applySettings
│
├── SECTION 19 · SGF Import / Export
│     buildSGF() — FF[4] serialiser
│     importSGF() — regex parser with error handling
│     copySGF() — clipboard API with fallback
│
├── SECTION 20 · Event Handlers
│     Canvas click/mousemove/mouseleave/touchend
│     document keydown (full shortcut suite)
│     Modal backdrop dismiss, window resize
│
└── SECTION 21 · Boot
      newGame()
```

---

## AI Design Details

### Search

White is the **minimising** player; Black is the **maximising** player. The root call iterates all of White's candidate moves, applies each, then calls `minimax(depth − 1, −∞, +∞, isMax=true)` for Black's response.

```
White (minimise)
└── Black (maximise)
    └── White (minimise)
        └── ...
```

Depth is calibrated so individual move response time stays below ~400ms on a mid-range laptop:

| Difficulty | 5×5 | 7×7 | 9×9 |
|------------|-----|-----|-----|
| Beginner   | random | random | random |
| Intermediate | 3 ply | 3 ply | 2 ply |
| Strong     | 6 ply | 5 ply | 4 ply |

### Evaluation Function

`evalBoard()` returns a signed value from Black's perspective:

| Component | Weight | Notes |
|-----------|--------|-------|
| Territory | ×2.8 | Flood-fill per empty region |
| Captures | ×2.2 | Differential (Black − White) |
| Komi | −komi | Subtracted directly |
| Atari (1 liberty) | −6.5 | Per group, signed |
| 2 liberties | −1.2 | Per group, signed |
| Liberty count | ×0.28 | min(libs, 6) per group |
| Stone count | ×0.14 | Per stone |
| Center control | ×0.07 | Weighted by proximity to center |

### Super-ko

Every board position is hashed into a 64-bit Zobrist key (two independent 32-bit values, XOR'd incrementally as stones are placed/removed). The set of all previously seen positions is maintained in `_posHistory` as a `Set<number>`. A move is rejected if the resulting hash key already exists in this set.

### Transposition Table

The TT stores `{ depth, val, flag }` keyed by Zobrist hash. Three flag types implement standard TT windowing:

- `'E'` — Exact value (returned directly regardless of window)
- `'L'` — Lower bound (returned if `val >= beta`)
- `'U'` — Upper bound (returned if `val <= alpha`)

The table is cleared at the start of each AI turn and evicted entirely when it exceeds 400,000 entries.

---

## SGF Format

Export produces a standards-compliant FF[4] SGF string:

```
(;FF[4]GM[1]SZ[9]KM[6.5]CA[UTF-8]AP[StoneGarden:4.0]
;B[pd];W[dp];B[pp];W[dd]...)
```

Import accepts any FF[4] SGF with `SZ[5]`, `SZ[7]`, or `SZ[9]`. Passes are encoded as `;B[]` / `;W[]`. The parser uses a single regex `(/;([BW])\[([a-s]{0,2})\]/g)` for robustness.


**Live deployment:** [https://go-game-ai.netlify.app/](https://go-game-ai.netlify.app/)

## Rules Reference

Stone Garden implements standard Go rules with Japanese scoring conventions:

- **Capture** — A group of stones with no liberties (empty adjacent intersections) is removed from the board after the opponent plays.
- **Ko** — Implemented as full super-ko: any board position that has occurred previously in the game is illegal, preventing all cycles not just simple ko.
- **Suicide** — Placing a stone that has no liberties after resolution is illegal, unless it captures at least one opponent group in doing so.
- **Scoring** — Territory (empty intersections surrounded exclusively by one colour) plus captures. Komi is added to White's total to compensate for Black's first-move advantage.
- **Game end** — Two consecutive passes from both players, board completely full, or resignation.

---

*Stone Garden · Go AI v4 · Single-file · Zero dependencies · Canvas 2D*
