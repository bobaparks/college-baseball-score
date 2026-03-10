# ⚾ Diamond College Baseball
### Live NCAA D1 Baseball Scoreboard

**https://www.diamondcollegebaseball.com**

A real-time NCAA Division I college baseball scoreboard built as a single HTML file, deployed on GitHub Pages, and accessible via a custom domain. Covers every D1 game across all conferences with live scores, box scores, individual player statistics, and in-game situational data.

Built entirely with AI assistance by someone with no formal coding background.

---

## What It Does

Diamond College Baseball pulls live data from ESPN's undocumented public API and presents it in a clean, fast, mobile-friendly scoreboard interface. Every D1 game is shown — not just the featured matchups — with real-time updates every 15 seconds.

---

## Features

### Live Scoreboard
- All NCAA D1 games for any date, browsable via a ±3 day date strip
- Scores update every 15 seconds without reloading the page
- Games organized into **Live Now**, **Upcoming**, and **Final** sections
- Team color accents pulled directly from ESPN's data

### Real-Time In-Game Situation
For live games, each card shows:
- **Base runners** — visual diamond with highlighted bases
- **Outs** — two dot indicators that fill in red
- **Ball/strike count** — displayed as B-S (e.g. 2-1)
- **Current batter** name
- Graceful **"Live data not yet available"** message when ESPN hasn't yet populated situation data for a game

### Box Score Modal
Clicking **BOX SCORE** on any completed or live game opens a modal showing:
- Score header with team records
- Full inning-by-inning linescore
- Individual **batting lines** with AB, R, H, RBI and more
- Individual **pitching lines** with IP, H, R, ER, BB, K
- Win/Loss/Save pitcher decisions
- Direct links to ESPN Gamecast and full ESPN box score
- Graceful **"Individual stats not yet available"** message for early-inning games

### Conference Filter
- Dropdown filters games by conference — SEC, ACC, Big 12, Big Ten, Sun Belt, and every other D1 conference
- Conference lookup uses a hardcoded map of ~200 team names since ESPN's scoreboard API returns no conference data

### HBCU Filter
- Dedicated **★ HBCU** filter showing only games involving HBCU programs
- Per-card HBCU badge on any team from the SWAC or MEAC conferences
- The only live D1 scoreboard with dedicated HBCU coverage

### Team Search
- Live text search filters cards by team name as you type

### Status Filter
- One-click filters for **All**, **Live**, **Final**, and **Upcoming** games

### Scores Ticker
- Scrolling ticker at the top showing all live and recent scores
- Pauses on hover

### Stats Leaders Ticker
- Second scrolling ticker below the scores ticker
- Shows national D1 leaders in **AVG**, **HR**, **RBI**, **ERA**, and **K**
- Toggle between **Overall D1** and **By Conference** views
- Seed data from d1sportsnet.com shown immediately on load
- Background refresh attempts live scrape via CORS proxy every 5 minutes

### Mobile Optimized
- Full responsive layout at 700px breakpoint
- Box score modal slides up from bottom on mobile

---

## Technical Architecture

### Single File
The entire application — HTML, CSS, and JavaScript — lives in one `index.html` file. No build tools, no dependencies, no npm, no framework. Deployed directly to GitHub Pages.

### ESPN API Endpoints
Two ESPN endpoints power the app:

**Scoreboard** (scores, linescore, situation, team data):
```
https://site.api.espn.com/apis/site/v2/sports/baseball/college-baseball/scoreboard?limit=200&dates=YYYYMMDD
```

**Game Summary** (box score, individual player stats):
```
https://site.web.api.espn.com/apis/site/v2/sports/baseball/college-baseball/summary?event={gameId}
```

### Smart DOM Patching
Rather than rebuilding the entire page on every 15-second refresh, the app:
1. Compares the new game list against the previous one
2. If game count and statuses are unchanged → **patches existing cards in place** (updates scores, bases, count, outs, linescore cells individually)
3. If any game changes status (scheduled→live, live→final) → **full re-render**

This eliminates the page flash on every refresh and makes the situational indicators feel genuinely real-time.

### CORS Proxy Fallback
The box score fetch tries the ESPN summary endpoint directly first. If the browser blocks it (CORS), it automatically retries through `corsproxy.io` before falling back to local scoreboard data.

### Conference Mapping
ESPN's scoreboard API returns zero conference information. A hardcoded JavaScript object (`CM`) maps ~200 team `shortDisplayName` values to their conference strings. Lookup is exact-match first, then case-insensitive fallback.

---

## What We Learned Along the Way

### The ESPN API has two different domains that return different data
`site.api.espn.com` and `site.web.api.espn.com` look almost identical but behave very differently. The `.api.` domain returns scoreboard data fine but strips out `boxscore.players` entirely from summary responses. The `.web.api.` domain returns the full player stat arrays. This single domain difference was the root cause of the box score showing errors for weeks.

### Inline onclick attributes are a trap
The original box score button used `onclick="openBoxScore('${id}','${awayName}','${homeName}')"`. Team names containing apostrophes (`St. Mary's`), parentheses (`Miami (OH)`), or ampersands (`Texas A&M`) silently broke the HTML attribute and caused the call to fail. The fix was switching to `data-gameid` attributes and a single delegated event listener — no string escaping needed.

### The loading spinner was causing the "stuck" bug
Every 15-second background refresh was overwriting `#main` with a loading spinner *before* the fetch ran. On any slow response from ESPN, the spinner just sat there permanently. The fix was a simple `scoresLoaded` boolean — show the spinner only on first load, refresh silently after that.

### Modal HTML position matters more than you'd think
Early in development the box score modal HTML was placed *after* the `</script>` tag. The script ran `getElementById('bsClose')`, got `null`, threw an error, and crashed the entire script — meaning no scores loaded at all. Moving the modal HTML to before the `<script>` tag fixed everything.

### ESPN's `situation` object isn't always populated
For games that just started or where ESPN hasn't yet sent situation data, `comp.situation` returns an empty object. Rendering bases/count/outs from an empty object produced a misleading display that looked like a scoreless game with nobody on base. The fix was checking for the presence of any situation field before rendering — and showing a "Live data not yet available" message instead.

### GitHub Pages doesn't support custom domains on subdirectory paths
A custom domain always maps to the root of a GitHub Pages site, not a subfolder like `/college-baseball-score/`. The solution is giving the app its own repository so it lives at the root.

### DNS propagation is just waiting
Connecting a Squarespace domain to GitHub Pages involves adding four A records pointing to GitHub's IPs and a CNAME for `www`. The `InvalidCNAMEError` GitHub shows is often just the DNS not having propagated yet — not an actual misconfiguration. With a 1-hour TTL, it resolves on its own.

### localStorage is per-device, per-domain
A hit counter built on `localStorage` only counts visits on the visitor's own browser. It resets when they clear cookies, switch devices, or — crucially — when the site moves to a new domain. It was measuring nothing useful and was removed.

### CORS proxies are a practical necessity for client-side scraping
The stats ticker scrapes live data from d1sportsnet.com. Browsers block cross-origin requests to third-party sites, so the fetch goes through `corsproxy.io` as a relay. This works well enough for non-critical background data but isn't reliable enough to depend on for primary content.

### Single-page JavaScript apps are harder for Google to index
Because all content is rendered client-side via `fetch()`, Google's crawler may only see the bare HTML shell when it visits. Meta tags, structured data (JSON-LD), a `sitemap.xml`, and Search Console submission help significantly — but there's no substitute for server-side rendering if SEO is the primary goal.

---

## SEO

The site targets a mix of head terms and long-tail keywords where the major sports sites have gaps:

- **Conference-specific searches** — `SEC baseball scores live`, `ACC baseball scoreboard`, `Big Ten baseball scores`
- **HBCU-specific searches** — `HBCU baseball scores`, `SWAC baseball scores`, `live HBCU baseball` — the most winnable niche
- **Feature-specific searches** — `college baseball live count balls strikes`, `college baseball base runners live` — unique to this app
- **Temporal searches** — `college baseball scores today`, `college baseball games today`

---

## Files in This Repo

| File | Purpose |
|------|---------|
| `index.html` | The entire application |
| `sitemap.xml` | Sitemap for Google Search Console |
| `robots.txt` | Search engine crawl instructions |
| `README.md` | This file |

---

## Credits

Built with the help of Claude (Anthropic) over many sessions of iterative development, debugging, and refinement. The ESPN API endpoints were documented by the community at [akeaswaran's gist](https://gist.github.com/akeaswaran/b48b02f1c94f873c6655e7129910fc3b) and [pseudo-r/Public-ESPN-API](https://github.com/pseudo-r/Public-ESPN-API).

---

*NCAA D1 Baseball · All Conferences · Real-Time · Free · No Ads*
