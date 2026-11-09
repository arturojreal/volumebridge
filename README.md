# VolumeBridge

An open, reusable toolkit for bridging physical devices, TouchDesigner, and Unreal Engine on an LED volume. It aims to give a virtual-production team a tested device-input hub, an OSC router, Unreal receivers, a look layer, operator controls, and a take-reset system, so a prop or a controller can drive a real-time scene and the operator can reset cleanly between takes.

Working name, change freely. Built and documented as a project through late 2026, and published generic so other stages with similar arrays can adapt it.

## What it does

- Audio-reactive analysis in TouchDesigner.
- Devices (MIDI controllers, gamepads, a Flipper Zero, a Pebble watch, Arduino or ESP32) sending and receiving over MIDI, serial, OSC, and HTTP/WebSocket.
- Unreal Blueprints receiving those inputs to drive geometry, materials, and Chaos physics.
- Video from Unreal back into TouchDesigner over Spout (same machine) or NDI (across machines) for a look or compositing layer, then out to a display or wall.
- Operator control via Stream Deck and Bitfocus Companion, with a one-button take reset.

## Design in one line

TouchDesigner is the hub: every prop and operator trigger flows into TD, which logs, debounces, remaps, and forwards to Unreal over OSC and Remote Control. No TouchEngine and no paid license required.

## Target stack

- Unreal Engine 5.7.4 (Windows), built-in OSC and Remote Control plugins.
- TouchDesigner 2025 series, Non-Commercial (1280x1280 cap, no TouchEngine).
- Spout for the same-machine video round trip (Off World Live plugin on the Unreal side); NDI as the cross-machine fallback.

## Repo map

- `docs/` planning and reference: research briefs, wall-time plan, toolkit architecture, and `runbooks/` session guides.
- `td/` TouchDesigner components and the `NAMING.md` conventions (component names, Custom Parameters, OSC address contract).
- `unreal/` Unreal 5.7 template project (OSC receiver, Remote Control preset, pushable physics, reset system).
- `firmware/` Arduino/ESP32 sketches (optional device path).
- `props/` Flipper and Pebble relay examples.
- `companion/` Stream Deck / Companion profiles.

## Status

v0.1 in progress, published gradually. The design is laptop-portable: everything can run on one Windows RTX 4090 laptop with Unreal and TouchDesigner co-located, so no LED volume or render cluster is required to use it. This public repo is a curated, generic edition, and more of the component set and session guides land here as they mature.

## License

To be set before the v0.1 publish (MIT suggested). See [docs/04-toolkit-architecture.md](docs/04-toolkit-architecture.md).

## A note on scope

This repo is generic by design. It contains no stage-specific hardware, addressing, or topology. If you adapt it to your volume, keep your stage config in a gitignored `private/` overlay on your own machine.
