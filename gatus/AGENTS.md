# Gatus / Overview

Adds to the root AGENTS.md. Verified metric traps (Gatus v5, 2026-10-06):

- **Lazily created counters:** every `gatus_results_*` counter (both `success` values, `connected`, each `code`) appears already at 1, so `increase()` drops the first event. Per hourly bucket `W = [$__interval] offset -$__interval`, count `(increase(x W) or min_over_time(x W) * 0) + ((min_over_time(x W) unless last_over_time(x[10m])) or min_over_time(x W) * 0)`. The left fallback keeps a series with one sample; the 10-minute lookback stops a brief scrape gap from counting a running total as new.
- **Config reloads re-register every metric:** series vanish and come back from 0 with the same labels, so a range-wide `increase()` misses an event when a counter returns at its old value. Range tiles sum the hourly buckets with `sum_over_time(bucket[$__range:1h])`, which also keeps them equal to the hourly charts.
- External (push) endpoints have `type="UNKNOWN"` and no `results_connected_total`; their duration is 0 unless the pusher sends `duration`. Response time and unreachable counts exclude them.
- `results_connected_total` only counts results that connected; unreachable checks are `results_total - results_connected_total`. A scrape gap at a bucket's start loses increments of counters that already existed, while a counter created in the gap counts in full, so that hour can overcount unreachable checks.
- `results_duration_seconds`, `certificate_expiration_seconds` and `endpoint_success` are gauges of the latest check only.
- A failed check records its timeout as its duration (10 s for ICMP). Response time keeps passing checks with `duration and endpoint_success == 1`.
- Every Gatus restart starts new series. Pool durations across pods with `sum_over_time / count_over_time`; a per-pod `avg_over_time` lets one slow check on a short-lived pod top the ranking.
- `resets(endpoint_success[R])` counts up-to-down transitions. An outage that begins with a Gatus restart isn't counted, because the new series starts at 0.
- The kube scrape adds an `endpoint` label (the port name); the dashboard's endpoint variable filters `name`.
