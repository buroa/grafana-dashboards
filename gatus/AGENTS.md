# Gatus / Overview

Adds to the root AGENTS.md. Verified metric traps (Gatus v5, 2026-10-06):

- **Lazily created counters:** every `gatus_results_*` counter (both `success` values, `connected`, each `code`) appears already at 1, so `increase()` drops the first event.
- **Config reloads re-register every metric:** series vanish and come back from 0 with the same labels, so `increase()` misses an event when a counter returns at its old value.
- **Count per minute instead:** sum `x - x offset 1m >= 0 or x < x offset 1m or (x unless last_over_time(x[10m] offset 1m))` over `[$__range:1m]` for tiles and `[$__interval:1m] offset -$__interval` for hourly charts. A series with no sample in the prior 10 minutes counts from its value; one that returns within 10 minutes of a reload loses its first event. Tiles then equal the sum of the hourly charts, scrape gaps keep their increments, and counts are whole numbers.
- External (push) endpoints have `type="UNKNOWN"` and no `results_connected_total`; their duration is 0 unless the pusher sends `duration`. Response time and unreachable counts exclude them.
- `results_connected_total` only counts results that connected; unreachable checks are `results_total - results_connected_total`. An HTTP check that gets a response also records its status code, so `results_code_total` cross-checks the split.
- `results_duration_seconds`, `certificate_expiration_seconds` and `endpoint_success` are gauges of the latest check only.
- A failed check records its timeout as its duration (10 s for ICMP). Response time keeps passing checks with `duration and endpoint_success == 1`.
- Every Gatus restart starts new series. Pool durations across pods with `sum_over_time / count_over_time`; a per-pod `avg_over_time` lets one slow check on a short-lived pod top the ranking.
- `resets(endpoint_success[R])` counts up-to-down transitions. An outage that begins with a Gatus restart isn't counted, because the new series starts at 0.
- The kube scrape adds an `endpoint` label (the port name); the dashboard's endpoint variable filters `name`.
