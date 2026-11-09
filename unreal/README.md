# unreal/ - Unreal Engine 5.7 template project

A generic template project with the receivers and systems the toolkit needs. No stage-specific content.

Plugins to enable (all ship with 5.7, none paid): Remote Control API, Remote Control Web Interface, OSC (Open Sound Control), Enhanced Input.

| Piece | Built in | Role |
|---|---|---|
| OSC receiver Blueprint | S9 | Receives TD OSC, drives actors and materials |
| Remote Control preset | S13 | Trigger surface: reveal, camera, reset |
| Pushable Blueprint + Input Mapping Context | S12 | Gamepad-driven Chaos physics |
| Spout sender setup | S17 | Sends the render to TD (Off World Live plugin) |
| reset system | S23 | Restores physics, camera, Sequencer, scene state |

Defaults: Remote Control HTTP on :30010, WebSocket on :30020. Keep the server on the LAN only. Test the 5.7.4 Passphrase-header behavior early (see research brief 1).

Build artifacts (Binaries, Intermediate, Saved, DerivedDataCache) are gitignored.
