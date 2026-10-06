# grafana-dashboards

Grafana dashboards for a home lab, published on [grafana.com](https://grafana.com/orgs/buroa/dashboards). Each folder holds the dashboard JSON, a README (also its grafana.com description) and screenshots of every row.

## Dashboards

| Dashboard | grafana.com | Data source |
| --- | --- | --- |
| [APC UPS / Overview](apc-ups) | [25867](https://grafana.com/grafana/dashboards/25867) | `snmp_exporter` with the `apcups` module |
| [brrpolice / Overview](brrpolice) | [25155](https://grafana.com/grafana/dashboards/25155) | [brrpolice](https://github.com/zariel/brrpolice) metrics |
| [cloudflared / Overview](cloudflared) | [25868](https://grafana.com/grafana/dashboards/25868) | cloudflared `--metrics` endpoint |
| [Envoy Gateway / Overview](envoy-gateway-overview) | [25869](https://grafana.com/grafana/dashboards/25869) | Envoy proxy metrics, cAdvisor |
| [Envoy Gateway / Upstream](envoy-gateway-upstream) | [25870](https://grafana.com/grafana/dashboards/25870) | Envoy proxy metrics |
| [Envoy Gateway / Downstream](envoy-gateway-downstream) | [25871](https://grafana.com/grafana/dashboards/25871) | Envoy proxy metrics |
| [qBittorrent / Overview](qbittorrent) | [25156](https://grafana.com/grafana/dashboards/25156) | [qui](https://github.com/autobrr/qui) metrics |
| [ZFS / Overview](zfs) | [24987](https://grafana.com/grafana/dashboards/24987) | node-exporter `zfs` collector, smartctl_exporter |

All of them use a single Prometheus data source (`DS_PROMETHEUS`). Each folder's `README.md` lists the exact metrics and variables.

## Install

**Grafana UI:** Dashboards → New → Import, enter the grafana.com ID and pick your Prometheus data source.

**grafana-operator:**

```yaml
apiVersion: grafana.integreatly.org/v1beta1
kind: GrafanaDashboard
metadata:
  name: zfs
spec:
  instanceSelector:
    matchLabels:
      dashboards: grafana
  datasources:
    - datasourceName: prometheus
      inputName: DS_PROMETHEUS
  url: https://grafana.com/api/dashboards/24987/revisions/9/download
```

## Conventions

Every dashboard follows the same layout so they read alike:

- **Title** is `<App> / <Page>`.
- **Top row** is six stat tiles that answer "is it healthy?" at a glance.
- **Open rows** show what you need day to day; **collapsed rows** (`<Topic> · <details>`) hold what you only need when something looks off.
- **Severity colors** run green → yellow → red.
- **Panel descriptions** are short and self-contained; they never refer to other panels by position.
- **Defaults** are the last 24 hours with a 1 minute refresh (ZFS: 6 hours, 30 seconds).
