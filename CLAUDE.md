# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ESPHome firmware for the water softener salt meter: an ESP32-C6FH4 (4 MB, `board: esp32-c6-devkitm-1`, `esp-idf`) reading
a DFRobot A02YYUW UART ultrasonic sensor, wired directly with no breakout PCB. Hardware, wiring,
calibration and the entity list are in `README.md`. ESPHome generates the firmware project into
`.esphome/` (gitignored build output, do not hand-edit).

## Commands

Local build tooling and the device address are machine-specific and live in `CLAUDE.local.md`
(gitignored). Read that first if you need to compile or flash.

- Validate config: `esphome config salt-meter.yaml`
- Compile only: `esphome compile salt-meter.yaml`
- Compile + flash (USB or OTA) + logs: `esphome run --device <port|device-ip> salt-meter.yaml`

There is no test suite. Correctness is `esphome config` + `esphome compile`, and on hardware the
`Sensor` connectivity entity and `Distanza Sensore`.

## Conventions to preserve

- Comments in English. Entity names stay in Italian where Home Assistant depends on them.
- **Entity names are load-bearing.** HA's salt automation uses `sensor.salt_meter_sale_rimanente_cm`,
  and the dashboard uses `sensor.salt_meter_sale_rimanente` (the % sensor). Renaming either, or the
  device `name: salt-meter`, breaks the automation silently. Coordinate with the HA config first.
- Don't put units or the device name in entity names: use `device_class` / `unit_of_measurement`.
- Tank geometry is in `number:` entities (defaults in `substitutions:`), never hardcoded in lambdas.
- The raw `a02yyuw` sensor stays `internal`. Filtering is median (spike rejection) before any
  averaging.
- Pins: avoid the C6 strapping pins (4, 5, 8, 9, 15), keep GPIO12/13 (USB-JTAG, the logger) and
  GPIO16/17 (UART0) free. The sensor is powered at 3.3 V: the C6 is not 5 V tolerant.
- `update_interval: never` templates need a `component.update` from `on_boot` (see `ota_partition`).
  `on_boot` is in list form.
- Guard pointers from ESP-IDF APIs before dereferencing.
- Restart goes through `btn_restart`, which marks the OTA image valid first (`safe_mode`).
