# CrowdSec / Overview

Adds to the root AGENTS.md. Verified metric traps (CrowdSec v1.8.1, helm chart 0.24.2, envoy-proxy-bouncer v0.8.2, 2026-10-20):

- **Relabelled labels:** the chart's ServiceMonitors write the node name into `machine`, so CrowdSec's own `machine` arrives as `exported_machine`; prometheus-operator's `endpoint` pushes the LAPI's to `exported_endpoint`. Never group by either; split LAPI calls by `route` from `cs_lapi_route_requests_total`.
- `cs_machines_last_heartbeat_timestamp` lives in LAPI memory and keeps every machine seen since the LAPI started. In Kubernetes each agent pod registers as a new machine, so replaced pods age forever. Not used.
- `cs_alerts` and `cs_active_decisions` are gauges the LAPI recomputes from its database on every scrape. `cs_alerts` is stored alerts per scenario, excluding CAPI; new alerts are its per-minute increases. A scenario missing a minute earlier counts from its value only if the same LAPI was scraped then, so a new pod or a first scrape never counts stored alerts as new.
- `cs_active_decisions` counts decisions, not IPs: an IP on two lists counts twice, so it runs above `bouncer_decision_cache_size`.
- **Lazily created counters:** `cs_appsec_rule_hits` (no `_total`), `cs_bucket_*`, `cs_node_wl_hits_*`, `cs_lapi_route_requests_total` and `bouncer_lapi_decisions_*` appear already above zero. Use `(increase(x[R]) or (min_over_time(x[R]) * 0)) + ((min_over_time(x[R]) unless x offset R) or (min_over_time(x[R]) * 0))`.
- `cs_appsec_*` carries the calling bouncer's IP as `source`, so every bouncer restart starts new series.
- `bouncer_requests_total{action}` is each request's final outcome, WAF included; IP-decision bans are `ban` minus `bouncer_waf_requests_total{action="ban"}`. Both start every action at 0.
- The bouncer's histograms use Prometheus default buckets (5 ms and up) and the LAPI's start at 5 ms, while real values are microseconds to a millisecond. Quantiles would be invented; use `_sum / _count`.
- The bouncer exports no `process_*` or `go_*` metrics.
- `cs_appsec_*` exists only with an `appsec` acquisition; `cs_papi_*` stays at 0 unless the LAPI is enrolled in the console with PAPI.
- Lines read have a per-datasource name (`cs_kubernetessource_hits_total`, `cs_filesource_hits_total`, …); `cs_parser_hits_total` counts the same lines for every datasource.
- Scenario and rule names repeat across namespaces (`crowdsecurity/x`, `custom/x`). Keep the full `reason` or `rule_name` and shorten into a separate `short` label, or the trimmed series collide and the panel goes blank.
