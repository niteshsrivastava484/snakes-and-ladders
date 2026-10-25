# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A browser Snakes & Ladders game. It is a single self-contained file, `index.html`, with inline CSS and JS and no dependencies. There is no build step, package manager, linter, or test suite.

## Run / check / deploy

- **Run locally:** open `index.html` in a browser (`start "" index.html` on Windows).
- **Syntax check** (the only automated check available; needs Node):
  ```bash
  sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/check.js && node --check /tmp/check.js
  ```
- **Deploy:** push to `main`. GitHub Pages serves the repo root at https://niteshsrivastava484.github.io/snakes-and-ladders/ and usually updates within 1–2 minutes. The file must stay named `index.html`, or the root URL returns 404.

## Git identity (personal repo; do not use FICO credentials)

- Remote: `git@github.com:niteshsrivastava484/snakes-and-ladders.git`. Push directly to `main`; there is no PR flow.
- Repo-local config already set: `core.sshCommand` pointing to `~/.ssh/id_ed25519_personal`, plus `user.name` and `user.email` set to the personal Gmail. Keep them. The global git identity and default SSH key belong to a FICO enterprise account, which can't push here.
- For GitHub API actions (repo settings, Pages), use the `github-personal` MCP. The `github` MCP is signed in to the FICO account.

## Testing

There are no automated tests. To test in a real browser, serve the folder (`python -m http.server 8765`) and open `http://127.0.0.1:8765/`, because the Playwright MCP blocks `file://` URLs. To test a specific situation, write a prepared `slGame` object to `localStorage` and reload. To force a die value, override `Math.random` (`() => (face - 1) / 6 + 0.01`). Use the Fast speed so tests run quickly.

## Architecture (all inside one IIFE in `index.html`)

- **State:** one serializable `game` object (`{ v, rules, jumps, players, turn, streak, over }`) holds everything needed to resume. `save()` writes it after every turn, and `loadGame()` validates it on load and throws away anything malformed. If you add game state, put it in `game`, add it to `loadGame()`'s validation, and bump `v` if old saves would no longer load correctly.
- **Board:** an SVG with `viewBox` 0–500. The layers are drawn in this order: cells → highlights → snakes/ladders → tokens. `center(n)` maps square `n` (1–100) to pixel coordinates in a zigzag: square 1 is bottom-left, and the row direction alternates every 10 squares. Every placement goes through `center()`.
- **Snakes and ladders:** these are data only. The active layout is `game.jumps` (`{ landOn: endUp }`). Entries pointing down are snakes and entries pointing up are ladders. `CLASSIC` is the default layout. `randomBoard()` generates a layout where no square is used twice (so jumps never chain), squares 1 and 100 are never used, total climb and total drop are within 25% of each other, and there are at most 3 snake heads in the top row. `drawFeatures()` redraws from `game.jumps` and keeps a reference to each feature's SVG group in `featureEls`, so `glow()` can highlight it.
- **Turn flow:** `roll()` is async: it animates the die, steps the token one square at a time, then applies a jump. `busy` blocks input while a move is animating. `gameId` goes up in `begin()` (shared by new and restored games), and each `await` checks `alive()`, so starting or restoring a game mid-move stops the old move cleanly. Computer seats roll through the `cpuTimer` set in `renderTurn()`.
- **Rules:** `game.rules.bounce` makes an overshoot count back down from 100; when it's off, you must land on 100 exactly. A 6 gives another roll. With `game.rules.threeSixes`, a third 6 in a row (tracked by `game.streak`) passes the turn without moving.
- **Speed:** `SPEEDS` holds every animation delay in JS. `applySpeed()` copies the step and slide timings into CSS variables, so the token transitions match.
- **Sound:** generated with the Web Audio API in `tone()`/`sfx`, so there are no sound files. The audio is only started after the first click or key press, because browsers block sound before that.
- **Keyboard:** Space/Enter roll only when no control has focus. The handler ignores those keys when the focus is on a `button`, `select`, `input`, `summary` or link.
- **Persistence (`localStorage`):** `slGame` (game in progress; removed when someone wins), `slSetup` (seats, names, rule options), `slRecord` (wins per name and the fastest win), `slSpeed`, `slMuted`. `slMode` is a key from an older version; `loadSetup()` reads it once, converts it and deletes it.
