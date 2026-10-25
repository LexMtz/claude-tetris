# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, no build, no tests, no linter. README and UI text are in Spanish.

## Running

Open `index.html` directly, or serve statically: `python -m http.server 8000` then visit `http://localhost:8000`.

## Architecture

Three files, one script, no modules: `index.html` (DOM + two canvases), `style.css`, `game.js` (all logic, global script scope, `'use strict'`).

- `game.js` holds game state in top-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...). Functions mutate them directly; there is no state object.
- `board` is a `ROWS × COLS` matrix. `0` is empty, `1–7` is an index into both `COLORS` and `PIECES`. Piece cells store their own type index, so `merge()` copies them straight into `board`.
- Game loop: `loop(ts)` via `requestAnimationFrame` accumulates `dropAccum` and gravity-drops when it passes `dropInterval`. `animId` is cancelled on pause, game over and `init()`. Resuming from pause must reset `lastTime`, or the first `dt` spikes.
- Piece flow: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` calls `endGame()` if the new piece collides.
- `clearLines()` owns level and speed updates: `level = floor(lines/10)+1`, `dropInterval = max(100, 1000 - (level-1)*90)`.
- Input is one `keydown` listener using `e.code`. `P` works while paused. Everything else is ignored when `paused || gameOver`.

## Gotchas

- Canvas size is hard-coded in `index.html` (`board` 300×600, `next-canvas` 120×120). Changing `COLS`, `ROWS` or `BLOCK` in `game.js` requires updating the canvas `width`/`height` to `COLS*BLOCK` × `ROWS*BLOCK`.
- `softDrop()` (keydown) and `loop()` gravity are separate code paths with duplicated collide-or-lock logic. Only soft drop awards points. Hard drop awards 2 per cell.
- Rotation (`tryRotate`) is `rotateCW` plus horizontal kicks `[0,-1,1,-2,2]` only. No SRS tables, no floor kicks, no counter-clockwise.
- `randomPiece()` is uniform random, no 7-bag.
- The `NEXT` preview uses the same `drawBlock` helper with its own 30px block size, independent of `BLOCK`.
