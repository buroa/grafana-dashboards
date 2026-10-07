# cloudflared / Overview

Adds to the root AGENTS.md. Verified metric traps:

- **Lazily created counters:** `response_by_code` appears already above zero, so `increase()` drops the first event. Use `increase(x[R]) or (min_over_time(x[R]) unless x offset R)`.
- `response_by_code` counts only after the full body is copied and never counts 101 upgrades.
- `request_errors` counts every proxy error: visitors cancelling (most of it) and unreachable origins alike. An unreachable origin is answered with a synthesised 502 that never reaches `response_by_code`, so the success rate cannot see an origin outage; the not-completed share can (cloudflared master, 2026-10-05).
- `quic_client_*` exists only with QUIC transport.
- `tunnel_register_fail` doesn't exist; use `rpc_client_failures`.
