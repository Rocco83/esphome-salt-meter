# Backlog

## Firmware

| # | Item | Notes |
|---|---|---|
| 1 | **Weak Wi-Fi at the tank** | −86 dBm on 2026-11-05 in place, on the C6FH4's PCB antenna. It works (2.5 h uptime from power-on), but it sits at the edge. Options: move the board out from behind the cabinet or metal, a closer AP, or a board with an external antenna (u.FL). Watch `WiFi Signal` and HA unavailability in history. |
| 2 | **"Pausa Misura" switch for cover removal** | Requested 2026-11-06. While on, the level entities hold their last value (nothing reaches HA statistics), so lifting the cover does not draw a spike to "empty". Switch auto-off after a timeout (e.g. 30 min, a number) so it cannot be forgotten on; restore off at boot. `Distanza Sensore` keeps updating, so the live distance can still be seen. |
| 3 | **Frozen-value detector** | 2026-11-06 00:11: 1255 consecutive frames at exactly 410 mm (spread 0), while the tank was full (~86 mm the evening before). A real echo has mm jitter: yesterday 81–94 mm. If N minutes of frames have zero spread, flag a `Sensor Bloccato` problem binary_sensor and stop publishing the level. Settle the cause first (power cycle / inspection, see below). |
| 4 | **Sensor power and mode under firmware control** | If #3 is a latch: (a) yellow wire to a GPIO held low = real-time mode, which bypasses the sensor's internal filter (our median already does the filtering); (b) power-cycle the sensor from the firmware when frozen. The sensor draws ~8 mA but more during bursts, so drive it through a small high-side switch (e.g. P-MOSFET), not straight from a GPIO; that contradicts "no PCB" a little, it is one transistor. |

## Open issue: 41.0 cm frozen reading (2026-11-06)

Raw frames constant at 410 mm (FF:01:9A:9A) for at least 2 minutes, zero spread, salt shown 16 cm / 33 % with the tank actually ~full. Not the firmware: the frame summary decodes the raw bytes. Board uptime 5.9 h, no reboot, Wi-Fi unchanged (−84 dBm). To try, in order:
1. Unplug the USB for 10 s (power-cycles the sensor too; a firmware restart does not, 3V3 stays on). Back at ~9–10 cm = sensor latch.
2. Still 41 cm: inspect the transducer for condensation or salt crust, and whether it points straight down, clear of the brine well and the walls.

## Done

- 2026-11-05 Calibration mode removed: `publish_interval` and `frame_summary_interval` back to 60s. The per-frame decoded log moved to VERBOSE (compiled out at the current DEBUG level).
