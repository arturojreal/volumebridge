# Stack and install list

Last verified: 2026-11-07. Pin target: Unreal Engine 5.7.4, TouchDesigner 2025 series (Non-Commercial), laptop-portable build.

All version and licensing claims below were checked against official docs on the date shown. Anything I could not confirm from an official source is tagged **[verify]**.

## Build decisions this list assumes

- **Where it runs:** everything is built and demoed on the Windows RTX 4090 laptop, with Unreal and TouchDesigner on the same machine so Spout (same GPU) carries the video round trip. The LED volume is a stretch showcase, not the build target. See `03-wall-time-plan.md`.
- **TouchDesigner tier:** Non-Commercial (free). Two consequences, both confirmed on Derivative docs: output is capped at 1280x1280, and TouchEngine (running TD components inside Unreal) is disabled. The pipeline in this toolkit uses OSC plus Spout/NDI instead, none of which need a paid tier. If you ever want TouchEngine or full-resolution output, an Educational license (USD 300, full Commercial features) is the upgrade path if you qualify, otherwise a Commercial license. Confirm your eligibility before relying on it.
- **Publishing:** the repo is generic. No stage-specific hardware, addressing, or topology goes in it. Stage-specific config stays in the gitignored `private/` folder on your own machine.

## Per machine

### Windows RTX 4090 laptop (primary build + demo rig)

| Software | Version | Why | License | Install |
|---|---|---|---|---|
| Unreal Engine | **5.7.4** | Pinned 5.7 series; match your production engine version | Free (Epic EULA) | Epic Games Launcher, https://www.unrealengine.com/download |
| TouchDesigner | Latest **2025** build | Audio analysis, OSC hub, look layer | Non-Commercial (free) | https://derivative.ca/download |
| Spout | Spout2 (current) | Same-machine GPU texture share UE to TD | Free, open source | https://spout.zeal.co |
| Off World Live Spout plugin (UE) | For UE 5.7 **[verify exact build on Fab]** | Gives Unreal a Spout sender/receiver | Free for non-commercial | Fab marketplace, search "Off World Live Spout" |
| NDI Tools + SDK | NDI 6.x | Cross-machine fallback when Spout cannot reach a target | Free | https://ndi.video/tools/ |
| Arduino IDE (or PlatformIO) | Latest 2.x | ESP32/Arduino firmware | Free | https://www.arduino.cc/en/software |
| USB serial drivers | CP210x and CH340 | ESP32/Arduino boards enumerate as COM ports | Free | Silicon Labs / WCH |
| Python | 3.12+ | Glue scripts (Flipper serial to OSC bridge) | Free | https://www.python.org |
| Bitfocus Companion | Latest 3.x **[verify]** | Stream Deck operator control | Free, open source | https://bitfocus.io/companion |
| qFlipper | Latest | Flipper firmware + serial access | Free | https://flipper.net/ (Downloads) |
| pebble tool / Core Devices app | Current | Build and push Pebble watch apps | Free, open source | https://developer.repebble.com, phone app store |

Controller software (install what matches your own MIDI controllers and gamepads, all free, versions **[verify]**):
- The **vendor driver and configuration utility** for any controller that is not class-compliant USB MIDI, or that needs a MIDI mode toggled on (many pad and groovebox controllers ship in a proprietary mode by default).
- The **vendor encoder/control-center app** for setting encoder modes (absolute mode is easiest for TouchDesigner to map).
- **Steam Input or DS4Windows**, only if Unreal does not detect your gamepads.

Unreal plugins to enable in-editor (all ship with 5.7, none paid):
- **Remote Control API** (Edit > Plugins > Messaging). HTTP on :30010, WebSocket on :30020 by default.
- **Remote Control Web Interface** (serves the browser control panel).
- **OSC (Open Sound Control)** (Edit > Plugins > Input Devices). Gives the OSC Server object and the `On OSC Message Received` / `On OSC Bundle Received` Blueprint events.
- **Enhanced Input** (default in 5.7) for the gamepad work.

### MacBook (research, patching, docs, firmware, profiles)

| Software | Version | Why | License | Install |
|---|---|---|---|---|
| TouchDesigner | Latest **2025** build | Build and test patches that do not need Spout | Non-Commercial (free) | https://derivative.ca/download |
| Syphon | Current | Mac equivalent of Spout (same-machine texture share) | Free, open source | http://syphon.github.io |
| NDI Tools (macOS) | NDI 6.x | Cross-machine video; works on Mac where Spout does not | Free | https://ndi.video/tools/ |
| Arduino IDE / PlatformIO | Latest | Firmware authoring | Free | as above |
| Bitfocus Companion | Latest 3.x | Build and export profiles | Free | as above |
| Python | 3.12+ | Glue + docs tooling | Free | as above |
| pebble tool | Current | Pebble app build | Free | as above |

Mac limits to plan around (confirmed as general platform facts, **[verify]** for 5.7 specifics): Unreal on macOS lacks hardware ray tracing and several virtual-production features, and Spout is Windows-only. So on the Mac you can build TouchDesigner patches, write firmware, draft docs, and author Companion profiles, but the Unreal plus Spout round trip is a 4090 task.

### Stage render nodes (reference only, not a build target)

Treated as a locked, genlocked production cluster you do not install onto for this project. The toolkit reaches the wall, if at all, as a video source into the stage's existing routing, not as software on those nodes. Class of machine: workstation GPUs with frame-lock sync cards and SDI capture, running the stage's own Unreal 5.7.x and session-control tooling. No toolkit install happens here without written change-control approval. See the risk register.

## What is deliberately not installed

- **TouchEngine-UE plugin:** needs a paid TD license. Off the table on Non-Commercial. The toolkit does not depend on it.
- **Any paid Spout/NDI bridge:** the free Off World Live plugin and built-in TD NDI/Spout TOPs cover the round trip.

## Sources

- [Unreal Engine 5.7 released (AWN, Nov 2025)](https://www.awn.com/news/unreal-engine-57-now-available)
- [Remote Control API WebSocket reference (Epic)](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-api-websocket-reference-for-unreal-engine)
- [OSC plugin overview (Epic)](https://dev.epicgames.com/documentation/en-us/unreal-engine/osc-plugin-overview-for-unreal-engine)
- [TouchDesigner Non-Commercial product page (Derivative)](https://derivative.ca/product/touchdesigner-non-commercial/77)
- [TouchDesigner Licensing (Derivative)](https://derivative.ca/UserGuide/Licensing)
- [TouchEngine-UE readme and releases (GitHub)](https://github.com/TouchDesigner/TouchEngine-UE)
- [Spout updates for 2026](https://spout.discourse.group/t/updates-for-2026/693)
- [NDI Tools](https://ndi.video/tools/)
- [Bitfocus Companion](https://bitfocus.io/companion)
- [Flipper Zero CLI docs](https://docs.flipper.net/zero/development/cli)
- [Pebble developer site (Core Devices)](https://developer.repebble.com/open-source/)
