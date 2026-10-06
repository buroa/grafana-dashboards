# Envoy Gateway / Upstream

Adds to the root AGENTS.md. Verified metric traps:

- **Lazily created counters:** per-code `envoy_cluster_upstream_rq{envoy_response_code}` and `upstream_rq_xx` appear already above zero, so `increase()` drops the first event. Use `increase(x[R]) or (min_over_time(x[R]) unless x offset R)`.
- `upstream_rq_time` runs from request complete to response complete, so streaming routes include transfer time and the tail is bimodal.
- `connect_timeout` is a subset of `connect_fail`.
