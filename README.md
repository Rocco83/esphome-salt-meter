# esphome-salt-meter

Salt level in the water softener's brine tank, for Home Assistant. An ESP32-C6 with a
DFRobot A02YYUW waterproof ultrasonic sensor in the lid measures the distance to the salt
surface. The firmware reports the salt height above the tank bottom (0 cm = empty) and a
percentage. ESPHome on `esp-idf`.

It replaces the previous meter (nodemcu-32s + HC-SR04-style `ultrasonic`, GPIO32/12) and keeps
its Home Assistant entities, see [Replacing the old meter](#replacing-the-old-meter).

| File | What it holds |
|---|---|
| `salt-meter.yaml` | board, network, sensor, level calculation, diagnostics |
| `secrets-sample.yaml` | copy to `secrets.yaml` (gitignored) |

## Hardware

- **MCU**: ESP32-C6FH4 board (4 MB flash in the chip), `board: esp32-c6-devkitm-1`, the 4 MB
  layout. It also runs on an 8 MB DevKitC-1, which just leaves flash unused.
- **Sensor**: DFRobot A02YYUW (SEN0311), 3–450 cm, 3 cm blind zone, IP67 probe, UART output.
- **Power**: USB into the DevKit. The sensor draws < 10 mA from the DevKit 3V3 pin.

No breakout PCB: the four sensor wires go straight to the DevKit header. The sensor is powered
at **3.3 V**, so its TX swings 0–3.3 V and needs no level shifter. Powered at 5 V it would put
5 V on a C6 pin, which is not 5 V tolerant.

### Wiring

| A02YYUW wire | DevKit pin | |
|---|---|---|
| red, VCC | 3V3 | **not** 5V, see above |
| black, GND | GND | |
| white, TX | GPIO6 | UART RX, 9600 8N1 |
| yellow, RX | not connected | internal pull-up = processed mode, ~100 ms frames |

GPIO6 is not a strapping pin (C6: 4, 5, 8, 9, 15) and leaves GPIO12/13 (USB-Serial-JTAG,
logs and flashing) and GPIO16/17 (UART0) free. To use another pin, change `sensor_rx_pin`.

Use Dupont crimp or solder + heatshrink on the sensor leads. The tank is humid, so keep the
DevKit outside the lid.

### Mounting

- Sensor face pointing straight down at the salt, centred, away from the brine well and the
  tank walls (the beam is ~40° wide).
- The face must stay **at least 3 cm above the highest salt level**. In the blind zone the
  sensor gives no valid reading (the firmware drops it, the level keeps its last value).

## Firmware

### Level calculation

```
distance  = median of ~5 s of frames, averaged over 60 s       ("Distanza Sensore")
height    = clamp(Distanza Fondo - distance, 0, Distanza Fondo - Distanza Pieno)
percent   = height / (Distanza Fondo - Distanza Pieno) * 100
```

`Distanza Fondo` (sensor face to tank bottom) and `Distanza Pieno` (sensor face to the
"full" level) are `number` entities, changeable from HA and stored in flash. The defaults
are in `substitutions:`: 49 cm (as on the old meter) and 0 cm. The scale therefore runs from
0 cm (empty) to the full depth of the tank. With `Distanza Pieno` at 0, 100 % is never quite
reached, because the sensor cannot see closer than 3 cm.

Calibration: with the tank empty, press **Calibra Fondo**. It copies the current
`Distanza Sensore` into `Distanza Fondo`. With salt in the tank, measure the depth with a
ruler and type it into `Distanza Fondo` instead.

The `a02yyuw` component validates each frame's checksum and drops readings ≤ 30 mm.

### Entities

| Entity | Content |
|---|---|
| `Sale Rimanente - cm` | salt height above the bottom, cm, one value a minute |
| `Sale Rimanente - %` | the same as a percentage of the usable depth |
| `Distanza Sensore` | filtered distance from the sensor face (diagnostic, for calibration) |
| `Distanza Fondo` / `Distanza Pieno` | geometry (config) |
| `Calibra Fondo` | sets `Distanza Fondo` from the current distance (config) |
| `Sensor` | connectivity: off = no valid frame for 10 s (cable, power, sensor fault) |
| `ESPHome Restart` | marks the OTA image valid first, so `safe_mode` does not count it as a failed boot |
| `Clear Stored States` | wipes flash preferences (geometry back to the defaults) and restarts |
| `Uptime`, `WiFi Signal`, `Free Memory`, `Chip Temperature`, `Reset Reason`, `OTA Partition`, `IP Address`, `MAC Address`, `ESPHome Version` | diagnostics |

Alerts (salt low) stay in Home Assistant, on `Sale Rimanente - cm`.

### First flash

1. `cp secrets-sample.yaml secrets.yaml` and fill in the Wi-Fi networks.
2. USB cable into the DevKit's native USB port (not the "UART" one), then
   `esphome run salt-meter.yaml` and pick the serial port.
3. In the log, `Sensor` turns on within a few seconds and `Distanza Sensore` publishes after ~5 s.
4. After that, OTA: `esphome run --device <device-ip> salt-meter.yaml`.

## Replacing the old meter

Home Assistant automations (salt alerts) and the dashboard use:

| entity_id | from |
|---|---|
| `sensor.salt_meter_sale_rimanente_cm` | `Sale Rimanente - cm` |
| `sensor.salt_meter_sale_rimanente` | `Sale Rimanente - %` |

Device name (`salt-meter`) and entity names are unchanged, byte for byte, so the entity_ids stay the
same. The old formula is the same too: height = 49 − distance, % = height / 49.

**But the new board has a new MAC, so new unique IDs.** ESPHome unique_ids are
`<MAC>/0/<type>/<Name>`. If the old entities still exist when the new board is adopted, HA assigns
`..._cm_2`. The salt automation then silently stops firing (it guards on `unavailable`/`unknown`).
The long-term statistics (since 2026-03-25) are keyed by entity_id, so they also continue only if the
ids are reclaimed. Order:

1. Flash the new board and check it on the bench (web UI on port 80).
2. **Power off the old meter** first, otherwise HA rediscovers it and step 3 undoes itself.
3. In HA, *Settings → Devices & services → ESPHome*, **remove the `salt-meter` integration entry**
   (not just the device). This frees every old entity_id.
4. Adopt the new `salt-meter`. Check that the two entity_ids above exist without a `_2`, that the
   salt automation resolves them, and that the history continues across the swap.

Changed from the old firmware, so these entity_ids change or disappear:

- `Uptime` is numeric (seconds). The old text uptime and `Salt Meter Uptime`
  (`sensor.salt_meter_salt_meter_uptime`, doubled stem) are gone.
- `WiFi Signal dB` → `WiFi Signal`. The auto wifi off/on below −80 dB is gone: ESPHome now
  roams on its own (`post_connect_roaming`).
- `Hostname` is gone (it is always `salt-meter`).
- `Restart` and `Clear Stored States` keep their names.
