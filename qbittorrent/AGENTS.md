# qBittorrent / Overview

Adds to the root AGENTS.md. Verified metric traps:

- qui tracker byte metrics are gauges. `rate()` spikes on torrent removal, so cap per-minute increments by the session counter.
