# Backlog

## Before production (cutover)

| # | Item | Notes |
|---|---|---|
| 1 | **`publish_interval` and `frame_summary_interval` back to 60s** | 5s only to make calibration quick. At 60s, `median` (25 frames, ~2.5 s) feeds `throttle_average` over a minute; the README's "one value a minute" assumes that. |
| 2 | **Remove or quiet the UART byte log** | `uart: debug:` logs every frame (~10 lines/s) at DEBUG. Drop the `uart.debug.log_hex` action (keep the counting lambda that feeds the frame summary) or remove the whole `debug:` block together with the frame counters and the `interval:`. |
