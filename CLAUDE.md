# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of self-contained browser games — each game is a **single HTML file** with all CSS and JavaScript embedded inline. No build tools, no dependencies, no server required. Open any file directly in a browser to play.

## Running the Games

```bash
# Open in default browser (Windows)
start shooter.html
start tictactoe.html

# Or via PowerShell
Invoke-Item shooter.html
```

## Git & GitHub Workflow

Always commit and push after meaningful changes:

```bash
git add <file>
git commit -m "concise description of what changed and why"
git push
```

Remote: `https://github.com/miracleyy03/claudecodetest`

## Architecture: shooter.html (SURVIVOR)

Single-file top-down shooter using HTML5 Canvas (900×600, scaled to window via CSS). All logic is plain JavaScript with no frameworks.

**Game state machine** — `gameState` string drives the entire app:
`MENU` → `PLAYING` → `WAVE_PAUSE` → `PLAYING` → `LEVEL_COMPLETE` → `PLAYING` → `VICTORY` / `GAME_OVER`

**Key global systems (all module-level variables, no classes):**
- `LEVELS[]` — wave definitions; each wave is a plain object `{grunt:N, runner:N, tank:N, boss:N}`
- `EDEFS` — enemy stat table (hp, speed, radius, damage, score, color)
- `player` object — single mutable object updated each frame
- `enemies[]`, `bullets[]`, `particles[]`, `floatTexts[]` — live entity arrays, filtered in-place each frame
- `spawnQueue[]` — shuffled list of enemy types yet to spawn for the current wave; emptying it sets `waveClearing=true`
- `announce` object — controls the centered overlay text (level/wave announcements)
- `stars[]` — starfield used on MENU / GAME_OVER / VICTORY screens

**Frame loop** (`loop` → `update(dt)` → `draw()`):
- `update` dispatches on `gameState` first; PLAYING branch runs player, spawner, enemies, bullets, collisions, particles
- `draw` always clears canvas, applies screen-shake translate, dispatches drawing, then draws scanline/vignette overlay on top
- Delta time is capped at 50ms to prevent physics tunnelling on tab-switch

**Drawing** — all characters are drawn with Canvas 2D primitives (`fillRect`, `arc`, `quadraticCurveTo`). Each enemy type has its own `draw*` function that `ctx.save/restore`s around a `ctx.translate + ctx.rotate(e.angle)` so the character always faces the player. The boss HP bar and tank HP bar are drawn **after** `ctx.restore` (unrotated, in screen space).

**Audio** — Web Audio API, procedurally synthesized. `ensureAudio()` must be called on first user gesture (it's called inside `startGame()`). All sounds are short oscillator → gain → destination chains created fresh per sound.

**Persistence** — high score stored in `localStorage` under key `'survivorHi'`.

## Architecture: tictactoe.html

DOM-based game (no canvas). Board is a CSS Grid of `.cell` divs. Game logic is ~60 lines; the minimax AI is a recursive function that evaluates all possible board states. Scores persist in JS variables only (reset on page refresh).
