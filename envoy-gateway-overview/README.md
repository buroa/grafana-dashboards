# Envoy Gateway / Overview

Envoy Gateway at a glance: status, traffic, status codes, response time, routes and the proxy's own cost.

[grafana.com/grafana/dashboards/25869](https://grafana.com/grafana/dashboards/25869) · import ID `25869`

![Overview](1-overview.png)

## Overview

The first stop for an Envoy Gateway data plane. The top row answers "is it healthy?": gateway status, success rate, request volume, share of requests under 100 ms, bytes served and open connections. Below it, server errors and slow requests, traffic, and the busiest and failing routes.

Collapsed rows hold the detail you only need when something looks off:
- **Responses · client errors:** 4xx responses per hour by status code.
- **Proxy · CPU, memory, heap, workers & stalls:** proxy CPU and memory, Envoy heap, overload heap pressure, worker balance and event loop stalls.
- **Config · updates, apply time & DNS:** xDS updates, config apply time, control plane connection and DNS timeouts.

Companion dashboards: **Envoy Gateway / Downstream** (client side) and **Envoy Gateway / Upstream** (backend side, per route).

## Requirements
- Envoy proxy metrics (`envoy_*`) scraped from the Envoy Gateway proxy pods, with a `pod` label.
- Prometheus 2.33 or later, for negative offsets.
- cAdvisor `container_cpu_*` and `container_memory_*` for the CPU and memory panels.

## Collapsed rows

### Responses · client errors

![Responses · client errors](2-responses-client-errors.png)

### Proxy · CPU, memory, heap, workers & stalls

![Proxy · CPU, memory, heap, workers & stalls](3-proxy-cpu-memory-heap-workers-and-stalls.png)

### Config · updates, apply time & DNS

![Config · updates, apply time & DNS](4-config-updates-apply-time-and-dns.png)
