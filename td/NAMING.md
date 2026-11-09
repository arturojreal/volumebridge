# Naming and conventions

Conventions for every TouchDesigner component in `td/`, and for the OSC addresses they send and receive. Unreal receivers and Companion profiles follow the same OSC contract.

## Files and components

- One component per `.tox`, saved in `td/`.
- File name matches the COMP name, lowerCamelCase, no spaces, no stage or venue names: `audioAnalysis.tox` holds the COMP `audioAnalysis`.
- The working host project is `td/host/volumeBridgeHost.toe`. It loads the components for development; the components themselves stay self-contained.
- No absolute paths. Reference inside a component with relative paths (`op('./audioIn')`, `parent()`), and reach outside only through Custom Parameters.

## Custom Parameters

- TouchDesigner requires parameter names that start with an uppercase letter followed by lowercase letters or digits: `Gain`, `Smoothing`, `Lowcrossover`. Use the label for readable text ("Low Crossover").
- Group parameters into pages:
  - **Settings**: what an operator tunes (gain, smoothing, thresholds).
  - **Io**: network and device settings (ports, hosts, device names, send rate).
  - **About**: `Version` (string, for example `0.1.0`) and `Notes`.
- Every parameter gets a sensible default, so the component works the moment it is dropped into a blank project.
- Ports, hosts, device names, and output resolution are always parameters, never hard-coded. Stage-specific values live in the gitignored `private/` overlay.

## Operators inside a component

- Inputs and outputs use In/Out operators named for what they carry: `in_audio`, `out_bands`, `out_env`.
- Internal operators get short descriptive names (`spectrum`, `bandSplit`, `envFollow`) instead of defaults like `audiospectrum1`.
- Add a Text DAT named `README` inside each component: what it does, its inputs and outputs, and its OSC addresses.

## OSC address contract

All addresses are lowercase, slash-separated, and namespaced by domain. Values are floats normalized to 0 to 1 unless noted.

| Namespace | Example | Meaning |
|---|---|---|
| `/audio` | `/audio/low`, `/audio/mid`, `/audio/high`, `/audio/env`, `/audio/beat` | Audio analysis output. `beat` is a 0/1 trigger. |
| `/device/<id>` | `/device/knobs1/enc1`, `/device/pad1/pad_3_5`, `/device/gamepad1/lx` | Normalized device input from the hub. |
| `/cue` | `/cue/reveal`, `/cue/next` | Story cues. Trigger value 1. |
| `/cam` | `/cam/move`, `/cam/preset` | Camera moves and presets. `preset` is an integer. |
| `/look` | `/look/enable`, `/look/mix` | Look layer controls. |
| `/reset` | `/reset/soft`, `/reset/hard` | Take reset. Trigger value 1. |
| `/sys` | `/sys/heartbeat`, `/sys/selftest` | Health and testing. |

### Device ids

Pick a short id per device, for example `pad1` for a pad controller, `knobs1` for an encoder surface, `gamepad1` to `gamepad4`, `pebble`, `streamdeck`. Keep your own device-to-id map alongside the hub.

Raw MIDI (notes, CCs, channels) stays inside the hub. Only normalized `/device` values and semantic addresses (`/cue`, `/cam`, `/look`, `/reset`) leave TouchDesigner, so swapping a controller means editing the hub's mapping table, not the Unreal side. Control names: `enc<n>` for encoders and knobs, `pad<n>` or `pad_<row>_<col>` for pads, `key<n>` for keys, and `lx`, `ly`, `rx`, `ry`, `l2`, `r2`, `cross`, `circle`, `square`, `triangle` for gamepads.

Add new addresses here first, then in the component and the Unreal receiver, so TouchDesigner and Unreal never drift. S10 turns this table into the full OSC spec with types and ranges.

## Versioning

- Each component carries its own `Version` parameter, bumped when its behavior or parameters change.
- Repo releases are git tags (`v0.1`, `v0.2`). Target `v0.1` by Dec 18.
- Keep a running toolkit list, and update it when a component first becomes usable or changes version.
