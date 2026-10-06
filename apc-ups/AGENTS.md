# APC UPS / Overview

Adds to the root AGENTS.md. Verified metric traps:

- snmp_exporter series split across exporter restarts; always aggregate with `max by (instance)`.
