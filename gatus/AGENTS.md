# Gatus / Overview

Adds to the root AGENTS.md. Verified metric traps (Gatus v5, 2026-10-06):

- **Lazily created counters:** `gatus_results_total{success="false"}` and each `gatus_results_code_total{code}` appear already at 1, so `increase()` drops the first event. Use `increase(x[R]) + ((min_over_time(x[R]) unless x offset R) or min_over_time(x[R]) * 0)`.
- External (push) endpoints have `type="UNKNOWN"` and no `results_connected_total`; their duration is 0 unless the pusher sends `duration`. Response time and unreachable counts exclude them.
- `results_connected_total` only counts results that connected; unreachable checks are `results_total - results_connected_total`.
- `results_duration_seconds`, `certificate_expiration_seconds` and `endpoint_success` are gauges of the latest check only.
- A failed check records its timeout as its duration (10 s for ICMP). Response time keeps passing checks with `duration and endpoint_success == 1`.
- Every Gatus restart starts new series. Pool durations across pods with `sum_over_time / count_over_time`; a per-pod `avg_over_time` lets one slow check on a short-lived pod top the ranking.
- `resets(endpoint_success[R])` counts up-to-down transitions. An outage that begins with a Gatus restart isn't counted, because the new series starts at 0.
- The kube scrape adds an `endpoint` label (the port name); the dashboard's endpoint variable filters `name`.
