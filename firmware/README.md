# firmware/ - Arduino and ESP32 sketches

Device firmware for the input hub.

| Sketch | Built in | Role |
|---|---|---|
| `basic_io/` | S6, S7 | Read button + sensor, two-way serial with TD |
| `osc_io/` | S8 | OSC over UDP both directions (ESP32/Wi-Fi) |

Serial format: comma-separated values, one line per frame. Prefer wired serial for permanent or show-critical links; Wi-Fi/OSC is for wireless convenience with a wired fallback. Install the CP210x/CH340 USB drivers so boards enumerate as COM ports.
