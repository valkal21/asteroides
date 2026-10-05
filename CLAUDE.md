# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla ES6. No dependencies, no bundler, no build step, no tests, no linter. UI text, comments and the README are in Spanish — keep new user-facing strings and comments in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder:

```bash
npx serve .   # then http://localhost:3000
```

## Architecture

All game logic lives in a single script, `game.js`, loaded by `index.html` (which only provides an 800×600 `<canvas id="canvas">`). The canvas size is duplicated as the `W`/`H` constants at the top of `game.js` — change both together.

- **Globals, not modules:** `canvas`, `ctx`, `W`, `H`, the `keys`/`justPressed` input maps, and the game state (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`) are top-level `let`/`const`. Entity `draw()` methods write to the global `ctx` directly.
- **Entity convention:** every entity class (`Bullet`, `Asteroid`, `Ship`, `Particle`) exposes `update(dt)` and `draw()`, has `x`, `y`, `radius` for circle collisions via `dist()`, and marks removal with `this.dead = true`. The main `update()` filters dead entities out of the arrays after each pass.
- **Toroidal space:** positions are wrapped with `wrap(v, max)` (bullets, asteroids, ship). Particles are the exception — they don't wrap.
- **Game loop:** `requestAnimationFrame(loop)` computes `dt` in seconds, clamped to 0.05 s. All speeds are in px/s, so any new movement must be multiplied by `dt`.
- **State machine:** `state` is `'playing' | 'dead' | 'gameover'`. `update()` branches on it at the top: `'dead'` counts down `deadTimer` then respawns the ship; `'gameover'` waits for Space to call `initGame()`. Level advances (`nextLevel()`) when `asteroids` is empty, spawning `3 + level` large asteroids away from the center (`SAFE_DIST`).
- **Input:** `keys[code]` is held state; `pressed(code)` is a consume-once edge trigger backed by `justPressed` (used for firing and restarting). Use `e.code` values (`'ArrowLeft'`, `'Space'`, …).
- **Asteroid sizes:** size is an index `3` (large) → `1` (small) into the parallel arrays `RADII`, `SPEEDS`, `POINTS`. `split()` returns two asteroids of `size - 1`. Adding a size or tuning scoring means editing those arrays.

## Note

The README mentions power-ups and special asteroid types (e.g. a shooting star), but these are not implemented in `game.js` yet.
