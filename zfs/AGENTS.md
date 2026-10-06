# ZFS / Overview

Adds to the root AGENTS.md. Verified metric traps:

- The ZIL on a special vdev is counted as "normal" in kstats, so SLOG counters at zero don't mean there is no SLOG.
