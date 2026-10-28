# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris in vanilla JS + HTML5 Canvas + CSS. No dependencies, no package.json, no build step, no linter, no test suite. UI text and README are in Spanish — keep user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve statically (preferred):

    python -m http.server 8000   # then http://localhost:8000

Verification is manual: play in the browser and check the console.

## Architecture

All logic lives in `game.js` (single script, `'use strict'`, global functions and module-level `let` state — no classes/modules). `index.html` supplies the DOM elements it grabs by id (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`); renaming an id requires changing both files.

- **Board**: `ROWS × COLS` matrix; `0` = empty, `1–8` = piece type, which doubles as index into `COLORS` and `PIECES`. Piece shapes store their type number in filled cells, so merged cells keep their color. Type 8 (`NUT`, tuerca) is a 3×3 ring with an empty center; its circle is drawn by `drawNutHole` (not stored in the board).
- **Pieces**: `{ type, shape, x, y }`; shape is a square matrix copied from `PIECES`. Rotation = `rotateCW` (transpose + reverse) with simple horizontal kicks `[0,-1,1,-2,2]` in `tryRotate` (not SRS).
- **Collision**: `collide(shape, ox, oy)` is the single check used for movement, rotation, gravity, ghost, and spawn (spawn collision → `endGame`). Cells with `y < 0` are allowed.
- **Loop**: `requestAnimationFrame` loop accumulates `dt` into `dropAccum`; when ≥ `dropInterval`, piece falls or `lockPiece()` (merge → clearLines → spawn). `draw()` redraws everything every frame. Pause/game over cancel the RAF via `animId`; resume restarts `loop` with a fresh `lastTime`.
- **Scoring/level**: `LINE_SCORES[cleared] * level`; soft drop +1/row, hard drop +2/row; `level = floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`. `updateHUD()` must be called after score changes.
- `init()` resets all state and is also the restart handler.

## Coupled constants

- `COLS × BLOCK` / `ROWS × BLOCK` must match `<canvas id="board" width height>` (300×600).
- `#next-canvas` is 120×120 = 4 cells × 30px (`NB` in `drawNext`), sized for the 4-wide I piece.
