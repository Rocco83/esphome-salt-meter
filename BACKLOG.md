# Backlog

## Firmware

| # | Item | Notes |
|---|---|---|
| 1 | **Weak Wi-Fi at the tank** | −86 dBm on 2026-11-05 in place, on the C6FH4's PCB antenna. It works (2.5 h uptime from power-on), but it sits at the edge. Options: move the board out from behind the cabinet or metal, a closer AP, or a board with an external antenna (u.FL). Watch `WiFi Signal` and HA unavailability in history. |
| 2 | **"Pausa Misura" switch for cover removal** | Requested 2026-11-06. While on, the level entities hold their last value (nothing reaches HA statistics), so lifting the cover does not draw a spike to "empty". Switch auto-off after a timeout (e.g. 30 min, a number) so it cannot be forgotten on; restore off at boot. `Distanza Sensore` keeps updating, so the live distance can still be seen. |
| 3 | **Frozen-value detector** | 2026-11-06 00:11: 1255 consecutive frames at exactly 410 mm (spread 0) with the tank full. **Cause: a broken cable**, not the sensor. A bad connection can therefore produce valid-looking frames at a constant value, which `Sensor` (frames arriving) does not catch. A real echo has mm jitter (81–94 mm the evening before). If N minutes of frames have zero spread, flag a `Sensor Bloccato` problem binary_sensor and stop publishing the level. |
| 4 | **Sensor power under firmware control** | Optional: power-cycle the sensor from the firmware when #3 trips. It draws ~8 mA but more during bursts, so use a small high-side switch (e.g. P-MOSFET), not a GPIO directly. One transistor, so slightly against "no PCB". |

## Done

- 2026-11-06 Frozen 41.0 cm reading: a broken cable. Fixed by rewiring; yellow moved to GND at the same time (real-time mode, no filtering inside the sensor).
- 2026-11-06 `Log Frame` switch: the decoded per-frame log, on demand from HA, off at boot. Replaces the VERBOSE-only log.
- 2026-11-05 Calibration mode removed: `publish_interval` and `frame_summary_interval` back to 60s.
