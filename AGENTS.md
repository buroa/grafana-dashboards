# AGENTS.md

House standards for the dashboards in this repo. Every dashboard follows them; a change that breaks one needs a reason.

## Layout

```
<dashboard>/
  dashboard.json     # exported dashboard, what grafana.com serves; gnetId = its grafana.com ID
  .lint              # dashboard-linter exclusions, each with a reason
  README.md          # grafana.com description + screenshots
  1-overview.png     # visible rows
  N-<row-slug>.png   # one per collapsed row, expanded, in row order
```

The root `README.md` has the index table (title, grafana.com ID, data source). Keep it in sync when adding a dashboard.

## Publishing

1. Edit the JSON. Keep `uid` and `gnetId` unchanged; grafana.com revisions and imports depend on them.
2. Verify (see below) and retake any screenshot whose row changed.
3. Open a pull request. The **Lint** workflow runs `dashboard-linter lint --strict` on every changed dashboard.
4. On merge to `main`, the **Release** workflow lints again and uploads each changed JSON to grafana.com as a new revision of its `gnetId`. Agents never upload by hand.
5. Renovate in `buroa/home-ops` bumps `revisions/<n>` in the `GrafanaDashboard` URL.

A new dashboard is created once by hand on grafana.com; then set its `gnetId` here.

## Design principles

The dashboard is a top-down story of plain questions. Accurate but busy charts get rejected.

- **Performance first.** Lead with what tells you whether the thing is healthy and fast.
- **Visible = needed + useful.** Everything moot goes into a collapsed row: flat lines, constants, usually-zero or always-empty panels, static facts, tuning internals, drill-down-only views, per-entity detail.
- **No duplicates**, within a dashboard or across the Envoy Gateway trio. A tile plus its time chart is fine; two charts of one measure, or two identical titles, are not.
- **One question per panel.** Within a row, the left panel answers the primary question; the right one gives attribution or context.
- **Works for anyone's setup.** Never reference another product, dashboard, host or cluster. Per-entity views must scale to 100 routes or 50 trackers: ranked top-10 bar gauges, plain tables, a drill-down variable. No by-name palettes.

## Structure

- **Dashboard title:** `<App> / <Page>`; single-page dashboards use `Overview`.
- **Top row:** six 4-wide stat tiles (title 13, value 30) that answer "is it healthy?". Tiles are colored by value, no background blocks.
- **Visible rows** are bare topic nouns (`Traffic`, `Cache`, `Disks`), ordered: is it healthy → what is it doing → why → cost/internals.
- **Collapsed rows** are `<Topic> · a, b & c`, reuse a visible topic noun where one fits, and are ordered by how likely someone opens them, mirroring the visible order.
- **Variable bar** stays on one line. Hide job/instance-style variables; drop a dashboard link if it wraps the bar.
- **Defaults:** last 24 hours, 1 minute refresh, `editable: false`, `graphTooltip: 1`, a single `DS_PROMETHEUS` input.

## Titles

- Sentence case, plain words, no units, no trailing punctuation. Panels use "and"; row titles keep "&".
- Breakdowns: `<measure> by <dimension>` ("Throughput by disk"), never "per disk".
- Bucketed counts: `<measure> per hour` / `per day`, never "Daily X", "Hourly X", "X history" or "X now".
- Rates: a standard noun (`Throughput`, `IOPS`) or `<measure> per second`.
- Tiles: one to three words, a noun phrase.
- The same measure has the same wording on every dashboard (`CPU`, `Memory`, `Throughput`).

## Descriptions

- At most about 20 words, plain language.
- Self-contained: never reference another panel, row or dashboard, by title or by position ("above", "see Memory"). That rots the moment something moves. Re-audit all descriptions after any move or removal.
- Range-based tiles say "in the selected range".

## Color

One meaning per hue, dashboard-wide and across dashboards:

| Hue | Meaning |
| --- | --- |
| Blue | served, read, download |
| Orange | received, write, upload, input |
| Purple | memory, ARC (MFU dark, MRU light) |
| Teal `#4fb3bf` | CPU (ZFS: L2ARC; APC: battery) |
| Gray `#8a8a8a` | neutral, limits (dashed), misses |
| Light gray `#c4c4c4` | secondary neutral |
| Green → yellow → red (dark red worst) | severity only |

- Orange is never a warning color.
- Status colors only on status.
- Shades of one hue only where the order is real (protocol versions, latency bands).
- Per-entity panels (disks, pods, replicas) use one flat color per direction.
- The same series has the same color in every panel.

## Panels

- **Forms:** hourly or daily bars for counts; 100%-stacked zone charts for shares; ranked bar gauges for who/what; tables for multi-attribute comparisons. Linear axes only.
- **Read/write pairs** go in one mirrored panel: write/upload below zero on the same axis.
- **Fills:** lines 0, single areas 15% gradient, stacked 50%, bars 85%. `lineWidth` 1. Filled areas `softMin` 0.
- **Stacked severity bands:** non-success bands get `lineWidth` 0, or the stacked outline turns red.
- **Legends:** lists at the bottom, never legend tables. Per-hour bars show a total (mean for percentages). One-line-per-entity panels get no legend; the tooltip names the line.
- **Tooltips:** multi, sorted descending.
- **Bar gauges:** basic display, value text 18; cap leaderboards at top 10.
- **Annotations:** toggles hidden, filtered to the panels they matter for.

**Never:**
- sparkline or trend columns in tables (tried and rejected)
- log axes
- percentiles of bimodal data
- short-window ratio lines
- big fonts on trivial values

Existing per-entity status-history grids (qBittorrent per tracker, ZFS per disk) are liked and stay. Don't add new ones where the entity count can explode (Envoy routes).

## Verification

Before calling a change done:

- **Queries:** run every query against a real Prometheus for each variable value; zero errors. Empty results only where by design. 7-day ranges should stay fast (under ~3 s).
- **Lint:** [`grafana/dashboard-linter`](https://github.com/grafana/dashboard-linter) with `--strict` must pass. Accepted exceptions live in each folder's `.lint` with a reason; keep them as narrow as the rule allows and never add one to silence a real finding.
- **Look at it:** import into a local Grafana, expand every collapsed row, screenshot the full page, and critique it as a reader would: nothing clipped, nothing confusing.
- **Screenshots:** before committing, blur anything private: tracker names, torrent names, personal data. Hostnames and bandwidth figures are fine.

## Metric traps

These are verified; don't relearn them.

- **Lazily created counters** (Envoy per-code `envoy_cluster_upstream_rq{envoy_response_code}`, `upstream_rq_xx`; cloudflared `response_by_code`) appear already above zero, so `increase()` drops the first event. Use `increase(x[R]) or (min_over_time(x[R]) unless x offset R)`. Envoy's downstream HCM `rq_xx` counters are pre-created and safe.
- **Envoy `upstream_rq_time`** runs from request complete to response complete, so streaming routes include transfer time and the tail is bimodal.
- **Envoy compressor `header_not_valid`** means the client's `Accept-Encoding` lists nothing Envoy offers.
- **Envoy `connect_timeout`** is a subset of `connect_fail`.
- **Prometheus 3 normalizes `le`** to `100.0`: match with `le=~"100(\\.0)?"`.
- **"Metric might not be a counter"** shows on `rate()` of any counter without a `_total` suffix. Harmless.
- **cloudflared:**
  - `request_errors` are mostly visitors cancelling, not origin faults.
  - `response_by_code` counts only after the full body is copied and never counts 101 upgrades.
  - `quic_client_*` exists only with QUIC transport.
  - `tunnel_register_fail` doesn't exist; use `rpc_client_failures`.
- **APC (snmp_exporter):** series split across exporter restarts; always aggregate with `max by (instance)`.
- **qui tracker byte metrics are gauges.** `rate()` spikes on torrent removal, so cap per-minute increments by the session counter.
- **ZFS:** the ZIL on a special vdev is counted as "normal" in kstats, so SLOG counters at zero don't mean there is no SLOG.

**Grafana 13 quirks:**
- A single-series bar gauge hides its name unless `displayName` is set.
- A state timeline with no data errors with "Data does not have a time field"; fall back with `or on() label_replace(vector(1), ...)`.
- The transpose transformation turns a leading string field into the header.
- Query variable regexes accept named `(?<value>…)(?<text>…)` groups.
