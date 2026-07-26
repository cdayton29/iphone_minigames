# Arcade

An offline-first PWA shelf for small games, built for an iPhone home screen. Pure HTML, CSS and vanilla JS — no build step, no dependencies.

## Files

```
index.html          hub shell + iOS meta tags
styles.css          hub styling (dark / light)
hub.js              reads games.json, paints the shelf, runs the stage
games.json          the registry — the only file you edit to add a game
manifest.json       PWA manifest
sw.js               service worker; caches the hub AND every game in games.json
icons/              app icons (180 / 192 / 512 / maskable)
games/
  solitaire/  index.html  script.js  icon.png
  sudoku/     index.html  script.js  icon.png
```

## Adding a game

**1. Make the folder.** Anything self-contained works:

```
games/minesweeper/
  index.html      ← the entry point
  script.js       ← cached automatically by folder convention
  icon.png        ← 256×256 works well
```

**2. Add one entry to `games.json`:**

```json
{
  "id": "minesweeper",
  "title": "Minesweeper",
  "subtitle": "Ten mines, no guessing",
  "icon": "games/minesweeper/icon.png",
  "path": "games/minesweeper/",
  "accent": "#7C9CFF"
}
```

`id`, `title`, `icon` and `path` are required. `subtitle` and `accent` (the colour of the card's spine) are optional. Two more optional keys:

- `entry` — if your entry file isn't `index.html`
- `assets` — extra files to cache: `["games/minesweeper/sprites.png", "games/minesweeper/audio.js"]`

**3. Bump the cache** in `sw.js`:

```js
const CACHE_NAME = 'arcade-v2';   // was arcade-v1
```

This is the step that makes iOS fetch the new files. Skip it and the phone will happily keep serving the old bundle forever. On next launch the hub shows a "New version downloaded" pill; tapping Reload swaps it in — never mid-game.

The hub reads `games.json` at launch and the service worker re-reads it at install, so nothing else needs touching. `hub.js` never has to change.

### What a game gets for free

- **Return to menu** — the floating Menu button lives in the hub, over the iframe, so it works no matter what the game does. A game can also request it: `parent.postMessage({ type: 'arcade:exit' }, '*')`.
- **Theme** — the hub passes `?theme=dark` or `?theme=light` on the URL and posts `{ type: 'theme', theme }` when it changes.
- **Deep links** — `.../#sudoku` opens straight into that game.

Keep the top-left ~120px of your game's UI clear; that's where the Menu button sits.

## Running it

Service workers need HTTP, so `file://` won't do. Locally:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

**Deploying:** push to a GitHub Pages repo or drag the folder onto Netlify. Everything resolves relatively, so project subpaths like `user.github.io/arcade/` work as-is.

**Installing on iPhone:** open the URL in Safari → Share → Add to Home Screen. Launch from the icon and the address bar is gone. Let it sit on the shelf for a moment on first launch so the service worker can finish caching, then it plays with no signal at all.

## The two sample games

**Solitaire** — Klondike, draw one. Tap a card to pick it up, tap a pile to drop it; double-tap sends a card to the foundations. Collect pulls up everything that's safe, Undo goes back 200 moves.

**Sudoku** — puzzles are generated on the device, with a uniqueness check on every dig, so each one has exactly one solution. Four levels, pencil marks, hints, and a Check that flags wrong entries for a couple of seconds.
