# World Cup 2026 — Live Tracker

A single-file web app that tracks the 2026 FIFA World Cup (USA · Canada · Mexico, 48 teams, 104 matches, June 11 – July 19) — live scores, FIFA-accurate group standings, a full knockout bracket, a Monte Carlo title forecast, and player/team stats. No backend, no build step, no API keys, no account. Just one HTML file you can host anywhere.

> Not affiliated with FIFA, Polymarket, or ESPN. Built as a personal project.

---

## What it is

The entire app is one self-contained `index.html` — HTML, CSS, and vanilla JS in a single file. Open it in a browser and it runs. Host it on GitHub Pages (or any static host) and the live features turn on. It pulls from free, public, keyless data feeds and computes everything else on-device.

If every network call fails, it still renders from a built-in snapshot, so the page is never blank.

---

## Features

### 🏠 HQ — the dashboard
- **Marquee match**: the next match today (with a kickoff countdown in your local time), or the latest result, or a live game in progress.
- **Pulse strip**: matches played, goals scored, groups decided, days to the Final.
- **Today's slate** and **recent results**.
- **Tournament leaders**: Golden Boot leader, market title favourite, biggest 24h odds mover.
- **Your bracket** progress and **Dallas (Arlington) watch** — countdown to the next match at AT&T Stadium.
- A path into the **full schedule** — all 104 fixtures, filterable by matchday and round, each showing its **venue** and your **device-local kickoff time**. Tap any match to enter a result manually.

### 📊 Standings
- **Group tables** computed live with the **official FIFA tiebreakers** (points → goal difference → goals scored → head-to-head → fair play → FIFA ranking).
- **Stats hub** with two views:
  - **Players** — Golden Boot (goals), penalty goals, assists, and disciplinary (yellow/red cards).
  - **Teams** — most goals, best defence, goal difference, clean sheets, most points, plus highlights (biggest win, highest-scoring match).

### 🏆 Bracket
Three modes:
- **Decide** — Round-of-32 slots lock in as each group finishes.
- **Project** — fills the whole bracket to a champion using market odds (then a consensus model), respecting real results.
- **Pick** — tap teams to advance them through to the Final and crown your own winner. Picks persist and cascade-invalidate correctly when an earlier pick changes.

The 8 best third-placed teams and their R32 slotting follow FIFA's combination table via an on-device constraint solver.

### 🔮 Forecast
- **Market odds** — live championship probabilities from a real-money prediction market, with 24h movement and the gap vs. a pre-tournament consensus model.
- **Simulation** — a Monte Carlo engine simulates the entire tournament 2K / 10K / 50K times using the real FIFA group engine plus a Bradley-Terry knockout model. Watch the bars converge live. Or **roll a single timeline** for one dramatic random tournament with a path-to-glory, upsets, a Cinderella run, and a chaos rating.

---

## Data sources

| Source | Provides | Notes |
|---|---|---|
| **[openfootball/worldcup.json](https://github.com/openfootball/worldcup.json)** | Schedule, results, goalscorers, venues | Free, public-domain, CORS-enabled. Updates roughly **daily** — it can lag a finished match by hours. |
| **Polymarket Gamma API** | Live championship odds | Keyless, CORS-friendly real-money prediction market. |
| **ESPN public API** (`site.api.espn.com`) | Live in-progress scores, yellow/red cards, assists, starting XIs | Keyless. Real-time. **Experimental** — see caveats below. |

**What's real vs. computed vs. manual:**
- **Real from the feed**: results, goalscorers, venues, kickoff times.
- **Computed on-device**: all standings, the bracket, the forecast, every team-stat leaderboard.
- **From ESPN (experimental)**: live scores, cards, assists, starting XIs.
- **Manual**: you can enter/override any match result, and log assists by hand. Both persist on your device.

---

## ⚠️ Honest caveats

- **ESPN features are unverified against a live response.** They're coded to ESPN's documented API shape and validated against synthetic payloads, but ESPN doesn't officially support cross-origin use, so **CORS may be blocked on some hosts**. Everything ESPN-related (live scores, cards, assists, XIs) is wrapped to fail silently — if it's blocked or the shape differs, those sections simply show no data and the rest of the app is unaffected. **Verify on your hosted site after a match finishes** (check Stats → Players and a team card). If empty, the ESPN field mapping may need a tweak.
- **openfootball lags real time.** For a result that hasn't landed yet, use the manual override (tap a match → enter the score). ESPN-final scores also auto-patch the lag when reachable.
- **Live in-progress scores are not counted in standings** until a match is final — by design.
- **The "Project" bracket is deterministic chalk** (always the favourite, no upsets). For probabilities, use the Monte Carlo forecast; for drama, roll a timeline.
- **Third-place R32 slotting** is a valid constraint-solver assignment that may differ from FIFA's official chart in rare edge cases until all 12 groups finish.
- **No possession / shots / xG** — those aren't in any free source used here.

---

## Setup & deployment

This is a static site. To deploy on **GitHub Pages**:

1. Commit `index.html` to your repo.
2. Enable Pages (Settings → Pages → deploy from your branch).
3. Visit `https://<username>.github.io/<repo>/`.

The browser **tab favicon is embedded inside the HTML** (base64 data URI), so it needs no extra file.

For the **iOS "Add to Home Screen" tile**, also commit `apple-touch-icon.png` (180×180) to the repo root — iOS looks for that filename automatically and is unreliable about reading icons from a data URI. For full PWA polish (Android home screen / splash), optionally add `icon-192.png` and `icon-512.png` and reference them from a `manifest.json`.

### Icon files in this repo
| File | Use |
|---|---|
| `apple-touch-icon.png` (180) | iOS home-screen tile |
| `favicon-32.png` / `favicon-16.png` / `favicon-48.png` | Classic favicons |
| `icon-192.png` / `icon-512.png` | PWA / Android |

---

## Local development

Just open `index.html` in a browser — the UI and the embedded snapshot work offline. **Live features (results sync, odds, ESPN) need a real web origin**, because browsers sandbox `file://` network requests and some feeds gate on origin. Host it (GitHub Pages, Cloudflare Pages, or a local static server like `python3 -m http.server`) to see live data.

No dependencies to install. No bundler. Fonts load from Google Fonts.

---

## Settings (⚙ in the header)

- **Auto-sync** every 5 minutes (off by default).
- **Live in-progress scores (ESPN)** — toggles all ESPN calls (on by default).
- **Show "vs predicted"** on group tables — overlays each team's pre-tournament model expectation.
- **Data source** and a manual **Refresh**.

Tapping the status pill in the header also refreshes. While any match is live, scores re-poll every 60 seconds on their own.

---

## How the data is stored

All state lives in the browser's `localStorage` — nothing leaves your device.

| Key | Contents |
|---|---|
| `wc26_overrides` | Your manual match results |
| `wc26_userpicks` | Your pick-bracket selections |
| `wc26_assists` | Your manually logged assists |
| `wc26_prefs` | UI preferences (active tab, sub-tabs, toggles, sim count, bracket mode) |
| `wc26_cache` / `wc26_sync` | Cached feed + last sync time |
| `wc26_poly` | Cached market odds |
| `wc26_live` | Latest ESPN live scores |
| `wc26_espndetail` | Harvested ESPN cards / assists per match |
| `wc26_lineups` | Latest ESPN starting XI per team |

To reset, clear the site's storage in your browser.

---

## Possible next steps

- A Cloudflare Worker proxy to bring sportsbook futures (FanDuel / DraftKings / Kalshi) live alongside the prediction-market odds.
- Share/export your pick-bracket as an image or URL.
- A `manifest.json` for full installable-PWA behaviour.
- Swapping ESPN for a paid stats API (API-Football / Sportmonks) if you want guaranteed cards, assists, lineups, and advanced metrics.

---

## License & disclaimer

Personal project. Match data is © its respective providers (openfootball is public-domain; Polymarket and ESPN data are used via their public endpoints). Not affiliated with, endorsed by, or sponsored by FIFA, Polymarket, or ESPN.
