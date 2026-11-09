# props/ - Prop relay examples

Bridges that turn a physical prop into a trigger the TD hub understands.

| Item | Built in | Role |
|---|---|---|
| `flipper_bridge.py` | S19 | Flipper Zero USB serial (or BLE) to OSC into the hub |
| `pebble/` | S20 | Pebble watch app + phone-side bridge to Remote Control / TD |

Principles: every prop trigger is debounced in the hub, has a wired path where possible, and has a manual backup button (see curriculum S24 and the risk register). The Pebble watch sends a simple message to its phone companion; the phone does the HTTP, so the watch app stays trivial. Confirm the current Pebble SDK networking API in coredevices/sdk-docs.
