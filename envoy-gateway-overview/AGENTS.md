# Envoy Gateway / Overview

Adds to the root AGENTS.md. Verified metric traps:

- **Lazily created counters:** per-code `envoy_cluster_upstream_rq{envoy_response_code}` and `upstream_rq_xx` appear already above zero, so `increase()` drops the first event. Use `increase(x[R]) or (min_over_time(x[R]) unless x offset R)`.
- The downstream HCM `rq_xx` counters are pre-created and safe with `increase()`.
