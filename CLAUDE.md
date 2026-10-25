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

## Architecture (all inside one IIFE in `index.html`)

- **Board:** an SVG with `viewBox` 0–500, drawn once at load. `center(n)` maps square `n` (1–100) to pixel coordinates in a zigzag: square 1 is bottom-left, and the row direction alternates every 10 squares. Every placement goes through `center()`.
- **Snakes and ladders:** these are data only, in the `SNAKES` and `LADDERS` maps (`{ landOn: endUp }`). Snake bodies are generated sine curves running from head to tail. Edit the maps and the drawing follows. A square must not appear as a key in both maps.
- **Turn flow:** `roll()` is async: it animates the die, steps the token one square at a time, then applies a snake or ladder. `busy` blocks input while a move is animating. `gameId` goes up on every `newGame()`, and each `await` checks it, so starting a new game mid-move stops the old move cleanly.
- **Rules:** a roll that would go past 100 doesn't move the token; you have to land on 100 exactly. A 6 gives another roll. In "You vs Computer" mode, `renderTurn()` triggers the computer's roll after a short delay.
- **Tokens:** each token is an SVG `<g>` moved with a CSS `transform`. They use the `slide` class for the slower snake/ladder animation. `OFFSETS` gives each player a fixed spot inside a square, so tokens on the same square don't overlap.
- **Persistence:** the browser's `localStorage` stores `slMode`, the selected player mode.
