# AGENTS.md

House standards for the dashboards in this repo. A change that breaks one needs a reason.

## Layout and publishing

```
<dashboard>/
  dashboard.json     # what grafana.com serves; gnetId = its grafana.com ID
  AGENTS.md          # verified metric traps for this dashboard (only if needed)
  .lint              # dashboard-linter exclusions, each with a reason (only if needed)
  README.md          # grafana.com description + screenshots; follow cloudflared/README.md
  1-overview.png     # visible rows, collapsed rows closed
  N-<row-slug>.png   # one per collapsed row, expanded, in row order
```

- Keep the root `README.md` index (title, grafana.com ID, data source) in sync.
- Never change `uid` or `gnetId`; revisions and imports depend on them.
- `dashboard.json` is `json.dumps(d, indent=2)` output: 2-space indent, non-ASCII escaped (`\u00b7` for `·`). Edit by loading and re-dumping, keeping the file's trailing newline or lack of one, so diffs stay surgical.
- A new dashboard is created once by hand on grafana.com; then set its `gnetId` here.
- Pull requests run **Lint** (`dashboard-linter lint --strict`). On merge, **Release** uploads each changed JSON as a new revision; agents never upload by hand. Renovate in `buroa/home-ops` bumps `revisions/<n>`.
- Commits are scoped to the dashboard folder or family: `fix(envoy-gateway): …`.

## Design

A top-down story of plain questions. Accurate but busy charts get rejected.

- **Performance first.** Lead with whether it's healthy and fast.
- **Visible only if someone checks it on a normal day.** Collapse the rest: flat lines, constants, usually-zero or empty panels, static facts, internals, drill-downs, per-entity detail.
- **No duplicates** within a dashboard or across the Envoy Gateway trio. A tile plus its time chart is fine; two charts of one measure or two identical titles are not.
- **One question per panel.** Visible rows usually pair the primary question (left) with attribution or context (right).
- **Works for anyone's setup.** No references to other products, dashboards, hosts or clusters. Per-entity views scale to 100 routes or 50 trackers (top-10 bar gauges, plain tables, a drill-down variable); no by-name palettes.

## Structure

- **Title:** `<App> / <Page>`; single-page dashboards use `Overview`.
- **Top row:** six 4-wide stat tiles answering "is it healthy?", colored by value, no background blocks. Apps with a state lead with an uppercase status word (ONLINE, CONNECTED, LIVE).
- **Visible rows:** bare topic nouns (`Traffic`, `Cache`, `Disks`), ordered: is it healthy → what is it doing → why → cost/internals.
- **Collapsed rows:** `<Topic> · a, b & c`, reusing a visible topic noun where one fits. The details name the panels inside, each idea once; rename the row when its panels change. Order by how likely someone opens them, mirroring the visible order.
- **Variable bar:** one line. Hide scrape plumbing (`job`, `instance` used only for scoping); entity selectors (host, gateway, route, tracker) stay visible. Drop a dashboard link if it wraps.
- **Defaults:** last 24 hours, 1 minute refresh, `editable: false`, `graphTooltip: 1`, one `DS_PROMETHEUS` input.

## Titles and descriptions

- Sentence case, plain words, no units, no trailing punctuation. Panels use "and"; row titles keep "&".
- Breakdowns: `<measure> by <dimension>`, never "per disk". Buckets: `<measure> per hour` / `per day`, never "Daily X", "X history" or "X now". Rates: a standard noun (`Throughput`, `IOPS`) or `<measure> per second`. Tiles: one to three words.
- One measure, one wording on every dashboard (`CPU`, `Memory`, `Throughput`).
- Descriptions: at most about 20 plain words, self-contained; never name another panel, row or dashboard, or point by position ("above"). Range-based tiles say "in the selected range".
- After any move or removal, re-audit every description and row title.

## Color

One meaning per hue, on every dashboard:

| Hue | Meaning |
| --- | --- |
| Text (default) | counts, ratios and facts with no direction or severity |
| Blue | served, read, download |
| Orange | received, write, upload, input |
| Purple | memory, ARC (MFU dark, MRU light) |
| Teal `#4fb3bf` | CPU (ZFS: L2ARC; APC: battery) |
| Gray `#8a8a8a` | neutral series, limits (dashed), misses, empty-state text |
| Light gray `#c4c4c4` | secondary neutral |
| Green → yellow → red (dark red worst) | severity and status only |

- Orange is never a warning; a neutral count is never green.
- Shades of one hue only where the order is real (protocol versions, latency bands).
- Per-entity panels use one flat color per direction; a series keeps its color in every panel.

## Panels

- **Forms:** hourly or daily bars for counts; 100%-stacked zone charts for shares; top-10 ranked bar gauges for who/what; tables for multi-attribute comparisons; status-history grids only for bounded entities (trackers, disks, IRC networks), never Envoy routes. Linear axes only.
- **Read/write pairs:** one mirrored panel; what the app sends out (reads, responses, a torrent client's upload) above zero, what it takes in below.
- **Fills:** lines 0, single areas 15% gradient, stacked 50%, bars 85%; `lineWidth` 1; filled areas `softMin` 0. Stacked non-success bands get `lineWidth` 0, or the outline turns red.
- **Legends:** lists at the bottom, never tables. Per-hour bars show a total (mean for percentages). One-line-per-entity panels get none.
- **Tooltips:** multi, sorted descending.
- **Usually-empty panels:** filter with `> 0`; `noValue` states the all-clear ("No errors", "All healthy").
- **Annotations:** toggles hidden, filtered to the panels they matter for.

**Never:** sparkline or trend columns in tables, log axes, percentiles of bimodal data, short-window ratio lines, big fonts on trivial values.

## Exemplars

Start from these panels instead of from scratch; they carry the house sizes and options.

| Need | Copy |
| --- | --- |
| Status tile | cloudflared · Tunnel |
| Count, rate and byte tiles | cloudflared top row |
| Hourly stacked bars | cloudflared · Requests per hour |
| Mirrored read/write | zfs · Client throughput |
| Per-day context | qbittorrent · Transfer per day |
| Top-10 ranked bars with drill-down | envoy-gateway-overview · Errors by route |
| Per-entity grid | qbittorrent · Upload by tracker per hour |
| Key/value facts | apc-ups · Battery record |
| Stats in a collapsed row | qbittorrent · Library |
| Usually-empty panel | brrpolice · Errors per hour |

## Verification

- **Lint:** `dashboard-linter lint --strict dashboard.json` passes. `.lint` exclusions are as narrow as the rule allows and never silence a real finding.
- **Queries:** run each one against the Prometheus in `$PROMETHEUS_URL` (ask if unset) for every variable value: no errors, empty only by design, 7-day ranges under ~3 s.
- **Look at it:** import into a local Grafana 13, expand every collapsed row and critique it as a reader: nothing clipped, nothing confusing.
- **Screenshots:** 1600 px wide, kiosk, dark; crop each collapsed row from its title to its last panel. Replace private names (trackers, torrents, personal data) with uniform gray pills; hostnames and bandwidth figures are fine.

## Metric traps

Verified; don't relearn them. Dashboard-specific traps live in that folder's `AGENTS.md`; note the exporter version a new one was verified on.

- **Prometheus 3 normalizes `le`** to `100.0`: match with `le=~"100(\\.0)?"`.
- **"Metric might not be a counter"** shows on `rate()` of any counter without a `_total` suffix. Harmless.

## Grafana 13 quirks

- A single-series bar gauge hides its name unless `displayName` is set.
- A state timeline with no data errors with "Data does not have a time field"; fall back with `or on() label_replace(vector(1), ...)`.
- The transpose transformation turns a leading string field into the header.
- Query variable regexes accept named `(?<value>…)(?<text>…)` groups.
