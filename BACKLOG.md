# Backlog

## Firmware

| # | Item | Notes |
|---|---|---|
| 1 | **Weak Wi-Fi at the tank** | −86 dBm on 2026-11-05 in place, on the C6FH4's PCB antenna. It works (2.5 h uptime from power-on), but it sits at the edge. Options: move the board out from behind the cabinet or metal, a closer AP, or a board with an external antenna (u.FL). Watch `WiFi Signal` and HA unavailability in history. |

## Done

- 2026-11-05 Calibration mode removed: `publish_interval` and `frame_summary_interval` back to 60s. The per-frame decoded log moved to VERBOSE (compiled out at the current DEBUG level).
