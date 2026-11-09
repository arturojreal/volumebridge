# Toolkit architecture

A generic, publishable toolkit for bridging devices, TouchDesigner, and Unreal on an LED volume. Working name: **VolumeBridge** (change freely). Nothing in the public repo is stage-specific; all stage-specific config lives in a gitignored `private/` overlay on your own machine.

## Design principles

1. **TouchDesigner is the hub.** Every prop trigger and operator action flows into TD, which logs, debounces, remaps, and forwards to Unreal. This is why: TD has OSC in and out at every license tier, Companion has a TD OSC module but no Unreal module, and a single hub gives one place to debug cross-app timing. (See research brief 7.)
2. **Loose coupling over TouchEngine.** The UE to TD link is OSC plus Spout/NDI, not TouchEngine. It needs no paid license, survives a crash on either side, and is easier to reason about. TouchEngine stays an optional future path gated on a paid license.
3. **Laptop-portable first.** Same-machine Unreal plus TD means Spout for the video round trip. NDI is the cross-machine fallback.
4. **Configurable, not hard-coded.** Ports, addresses, device names, and resolutions are parameters or config, so another stage adapts by editing config, not internals.
5. **Fail safe.** Every wireless or prop trigger has a manual backup. The reset system returns a scene to a known state between takes.

## Repo structure

```
/                      repo root (this folder)
  README.md            overview, quickstart, the generic pitch
  LICENSE              MIT suggested (confirm before publishing)
  .gitignore           excludes private/ and build artifacts
  docs/                the planning and reference docs
    01-research-briefs.md
    03-wall-time-plan.md
    04-toolkit-architecture.md
    runbooks/            step-by-step session guides
  td/                  TouchDesigner components (.tox) + a host project
    audioAnalysis.tox
    deviceInputHub.tox
    oscRouter.tox
    operatorPanel.tox
    lookLayer.tox
    remoteControl.tox
    NAMING.md
  unreal/              Unreal 5.7 template project
    OSC receiver Blueprint
    Remote Control preset
    Pushable Blueprint + Input Mapping Context
    reset system
  firmware/            Arduino/ESP32 sketches
    basic_io/
    osc_io/
  props/               prop relay examples
    flipper_bridge.py
    pebble/
  companion/           Stream Deck / Companion profile exports
  private/             gitignored: stage-specific config, never published
```

## Components (what each reusable piece does)

| Component | Where | Role |
|---|---|---|
| `audioAnalysis.tox` | td | Bands + envelope + beat from live audio, parameterized |
| `deviceInputHub.tox` | td | Normalizes MIDI, gamepad, serial, and OSC device inputs, debounced |
| `oscRouter.tox` | td | Namespaced OSC addressing, rate limiting, remap table |
| `remoteControl.tox` | td | Talks to Unreal Remote Control over HTTP/WebSocket |
| `operatorPanel.tox` | td | Operator UI for cues, camera, look toggles |
| `lookLayer.tox` | td | Color/comp layer applied to the Unreal feed |
| OSC receiver BP | unreal | Receives TD OSC, drives actors and materials |
| Remote Control preset | unreal | The trigger surface (reveal, camera, reset) |
| Pushable BP + IMC | unreal | Gamepad-driven Chaos physics interaction |
| reset system | unreal | Restores physics, camera, Sequencer, scene state |
| `flipper_bridge.py` | props | Flipper serial/BLE to OSC |
| pebble app + bridge | props | Watch to phone to Remote Control/TD |
| Companion profile | companion | Stream Deck operator control via TD |

## Signal flow (laptop build)

```
Props (Flipper, Pebble, Stream Deck)  ->  TouchDesigner hub  ->  Unreal
  MIDI controllers and gamepads                          ->  deviceInputHub
  Flipper (optional): USB serial / BLE -> flipper_bridge.py -> OSC -> deviceInputHub
  Pebble:  watch -> phone -> HTTP/WS                     ->  (Remote Control) / hub
  Companion: OSC (TD module)                             ->  hub
Audio in -> audioAnalysis -> oscRouter -> OSC            ->  Unreal OSC receiver
Unreal render -> Spout -> TouchDesigner lookLayer -> output display (wall stand-in)
Reset: one button -> hub/Companion -> Remote Control -> Unreal reset system
```

## Naming and versioning

- Components: lowerCamelCase `.tox` names matching their COMP name, no spaces, no stage names.
- Parameters: grouped pages, sensible defaults, every externally useful value promoted.
- Versioning: semantic-ish per component in its Custom Parameters (a `Version` string), plus repo git tags (`v0.1`, ...). Target v0.1 by Dec 18.
- Config: a single `private/config` the components read for ports, addresses, device names, and output resolution. Public defaults ship in the components; the private overlay overrides them.

## Licensing options (confirm before publishing)

- **MIT** (suggested): simplest, permissive, good for broad reuse by other VP teams.
- **Apache 2.0:** permissive with an explicit patent grant.
- **GPLv3:** copyleft, if you want derivatives kept open.
MIT is the lightest-weight choice for a toolkit meant to spread. Decide in S25 and add the `LICENSE` file before the v0.1 publish.

## What stays private

Stage hardware, addressing, topology, port assignments, host names, and anything derived from a specific venue's internal documentation. These live only in the gitignored `private/` overlay and in your separate local work folder, never in the repo. Any venue under a confidentiality agreement stays out of this public repo entirely.
