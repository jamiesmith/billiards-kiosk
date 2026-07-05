# 🎱 Billiards Kiosk

A single-page scorekeeper for pool/billiards, built to run on a tablet mounted (or propped up) next to the table. Pick a game, pick your players, and let the tablet handle scores, shooting order, and who's up next while everyone else just plays.

Live at [billiards.grandelatte.com](https://billiards.grandelatte.com).

## What it does

- **Game types** — 8-Ball, 9-Ball, 10-Ball, and Straight Pool, each with their own scoring UI (solids/stripes tracking for 8-Ball, a running point total with race-to targets for Straight Pool).
- **Teams or singles** — put one or two players per side; with two, the app tracks shooting order and alternates it after every win.
- **Play Mode** — a full-screen, landscape-friendly scoreboard meant to be read from across the room, with a big "Win" button per side and a coin flip to decide who breaks.
- **Winner-stays-on queue** — add players to an "up next" queue mid-session; when a game ends, the loser's spot is backfilled by the next name in line.
- **Session summary** — tracks games won per side across a session and shows a final tally when you end it.

## How it works

This is a Jekyll site with no build tooling beyond Jekyll itself — there's no separate frontend framework or bundler. Almost everything lives in [index.html](index.html):

- The player roster is injected at build time from [_data/billiards.yml](_data/billiards.yml) via `{{ site.data.billiards.players | jsonify }}`, so Jekyll bakes the names directly into the page as a JS array (`BASE_PLAYERS`).
- All UI state lives in a single `S` object (current game type, teams, scores, queue, etc.), and every user action funnels through one dispatch function, `D(action, a, b)`, which mutates `S` and re-renders.
- Rendering is plain template strings (`viewSetup()`, `viewGame()`, `viewSummary()`) re-assigned to `#app`'s `innerHTML` — no virtual DOM, no diffing.
- Adding a new game type means adding an entry to the `GAME_TYPES` registry (ball icon, extra state, and any custom scoring UI) — the setup/game/queue flow around it is shared.

Because it's just static HTML/CSS/JS produced by Jekyll, it's well suited to running unattended on a kiosk-mode tablet: no server-side state, no login, nothing to keep alive besides the browser tab.

## Running it locally

Requires Ruby (see [.ruby-version](.ruby-version) — 3.4) and Bundler.

```bash
bundle install
./runit.sh
```

`runit.sh` starts `jekyll serve` bound to your machine's LAN IP (instead of `localhost`) so you can load the page from a tablet on the same network while developing.

Pushes to `main` auto-deploy to GitHub Pages via [.github/workflows/jekyll.yml](.github/workflows/jekyll.yml).

## Forking it for your own group

If you want to run this for your own pool table, the only file you need to change is [_data/billiards.yml](_data/billiards.yml):

```yaml
players:
  - Alice
  - Bob
  - Chris
```

Add or remove names as needed — each one becomes a selectable player on the setup screen. Anyone not in the file can still be added on the fly as a one-off ("ad hoc") player from the kiosk itself, so the YAML file only needs your regulars.

### Overriding the roster without editing the YAML

You can also swap in a roster for a single page load with a `?players=` query string, no rebuild required:

```
https://billiards.grandelatte.com/?players=Alice,Bob,Chris,Dana
```

Names are comma-separated; use `+` or `%20` for spaces in a name (e.g. `?players=Jamie+Smith,Amy,Bob`). This is handy for letting someone try the kiosk with their own group's names without forking the repo — it only affects that one page load and falls back to `_data/billiards.yml` when the parameter is omitted.

A few other things worth updating if you fork this:

- [_config.yml](_config.yml) — `title`, `description`, and `url` (the `url` matters if you deploy to GitHub Pages).
- [CNAME](CNAME) — only needed if you're pointing a custom domain at GitHub Pages; delete it if you're using the default `*.github.io` URL.
- The banner text and "Pool House Rocks" name in the `BANNER` constant near the bottom of [index.html](index.html), if you want to rebrand the on-screen title.

No other code changes are required — the app reads the roster fresh from the YAML at build time.
