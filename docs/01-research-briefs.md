# Research briefs

Each brief is a short, sourced answer to one item in the project research queue. All sources checked 2026-11-07. Claims not backed by an official source are tagged **[verify]**. Where a detail is specific to UE 5.7.4, it is called out, because point-release behavior changes.

---

## 1. Unreal Remote Control API (HTTP and WebSocket, web panels)

**What it is:** a built-in plugin that lets external apps read and set exposed properties and call functions over HTTP and WebSocket. It is the cleanest path for a watch, a phone, Companion, or TouchDesigner to trigger presets in Unreal.

**Setup:** enable **Remote Control API** and **Remote Control Web Interface** under Edit > Plugins (Messaging category). HTTP server defaults to port **30010**, WebSocket to **30020** (`ws://127.0.0.1:30020`). Both ports are configurable in Project Settings. The server starts when the project is open in a supported mode (Editor, PIE, or game). Messages are JSON with a `MessageName`, `Parameters`, and an optional `Id` to match responses. You expose properties and functions by adding them to a **Remote Control Preset**, then hit them from the web panel or your own client.

**5.7.4 caveats [verify hands-on]:** one forum report says 5.7.4 crashes unless a `Passphrase` header is sent, and that `/remote/batch` works over HTTP but not over WebSocket while `/remote/preset` and `/remote/info` work over WebSocket. Treat as a single user's report; confirm on your build early, because the capstone depends on this transport.

**Security:** Epic states the server is for LAN or a secure VPN only, never exposed to the open internet. On stage this matters: keep it on the control network, not anything routable.

**Sources:** [Remote Control for UE (Epic)](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-for-unreal-engine), [WebSocket reference (Epic)](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-api-websocket-reference-for-unreal-engine), [5.7.4 batch-over-WebSocket forum thread (2026)](https://forums.unrealengine.com/t/can-the-remote-batch-endpoint-be-used-via-websocket-remote-control-in-unreal-5-7-4/2725878).

---

## 2. OSC in Unreal (plugin setup and Blueprint receivers)

**What it is:** Unreal ships an **OSC (Open Sound Control)** plugin. No third-party plugin needed.

**Setup:** Edit > Plugins > Input Devices, enable **OSC (Open Sound Control)**. Create an **OSC Server** (`UOSCServer`), a listening endpoint for messages sent to the local instance, with multicast loopback and parsing of both messages and bundles. In Blueprints, bind **On OSC Message Received** and **On OSC Bundle Received**. An OSC message has an address (a valid path) and a typed payload; Blueprint nodes read values by type. You set the listen IP and port on the server, and can allowlist clients and bind address patterns to filter traffic.

**Why it carries the bridge:** TouchDesigner has OSC In/Out built in at every tier, so TD to Unreal over OSC needs nothing paid on either side. This is the spine of parts 2, 3, and 7 of the curriculum.

**Sources:** [OSC plugin overview (Epic)](https://dev.epicgames.com/documentation/en-us/unreal-engine/osc-plugin-overview-for-unreal-engine), [OSCServer Python API, 5.7](https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/OSCServer?application_version=5.7).

---

## 3. Spout and NDI for Unreal (options, latency, versions)

**Architecture:** Spout shares a GPU texture between apps on the **same machine** (sender and receiver must be co-located). NDI encodes frames on the CPU and sends them **over the network**, so it crosses machines at the cost of more latency and CPU.

**For this build:** Unreal and TouchDesigner run on the one 4090 laptop, so **Spout is the round-trip path** (lowest latency, best quality for local interop). NDI is the fallback only when a target is on another machine, for example feeding the wall from a second box. Current tooling uses **NDI 6.2.1**; an emerging open protocol called **OMT** is worth watching for low-latency LAN video but is not needed here.

**Unreal side:** Unreal does not ship Spout. Use the **Off World Live Spout plugin** (free for non-commercial on Fab) for a Spout sender/receiver, or go NDI with the NDI plugin. TouchDesigner has Spout In/Out TOPs (Windows) and NDI In/Out TOPs built in.

**Latency numbers:** no published 2026 benchmark for UE or TD was found. Measurements are anecdotal (Spout faster, NDI heavier). Benchmark on your own hardware if a number is needed. **[verify by measurement]**

**Sources:** [Spout updates for 2026](https://spout.discourse.group/t/updates-for-2026/693), [Off World Live Streaming Toolkit noted in TD+UE integration write-up (Medium)](https://medium.com/@Jamesroha/touchdesigner-and-unreal-engine-5-integration-ebb5b12d0609).

---

## 4. TouchEngine for Unreal (setup and limits)

**What it is:** the TouchEngine-UE plugin loads TouchDesigner components **inside** Unreal via the TouchEngine API, passing Custom Parameters, CHOP, TOP, and DAT in and out. Current release: plugin **v1.7.0**, built for **UE 5.7.1**, needing **TouchEngine 2025.31760+**. Earlier releases target 5.6.x, 5.5.x, 5.4.1 and older, so match the build to your engine from the releases page.

**The blocker for this project:** TouchEngine **does not work with a Non-Commercial TD license** (stated in the plugin README and Derivative's licensing table). It needs an installed paid license (Commercial, Pro, or Educational) or TouchPlayer.

**Decision:** TouchEngine is out of scope while on Non-Commercial. The toolkit's UE to TD integration uses OSC plus Spout/NDI, which is arguably the better fit for a live round trip anyway (looser coupling, easier to debug, survives a crash on either side). Revisit TouchEngine only if an Educational or Commercial license is acquired. A third-party post calls the plugin Windows-only; **[verify]** against Derivative docs before relying on that.

**Sources:** [TouchEngine-UE readme + releases (GitHub)](https://github.com/TouchDesigner/TouchEngine-UE), [TouchEngine for UE user guide (Derivative)](https://derivative.ca/UserGuide/TouchEngine_For_Unreal_Engine_Plugin), [Licensing (Derivative)](https://derivative.ca/UserGuide/Licensing).

---

## 5. Pebble: sending HTTP requests from a watch app

**Current status:** Pebble is revived under **Core Devices**, firmware and SDK and tools are open source, repos at https://github.com/coredevices, developer site at developer.repebble.com, and a **pebble** command-line tool builds, installs, and debugs apps and manages SDKs. There is a current phone companion app (Core Devices app, on the stores).

**HTTP path [verify current SDK]:** the search did not surface current networking docs. Historically a Pebble watch app does not make raw HTTP itself; it talks to a phone companion over a message channel (classic era: AppMessage plus PebbleKit JS on the phone, which does the HTTP). The current SDK's networking API should be confirmed in the **coredevices/sdk-docs** repo. The practical relay for this project does not depend on that detail: watch app to phone companion to HTTP or WebSocket into Unreal Remote Control (or into TouchDesigner, which then forwards). Build the phone-side bridge so the watch only needs to send a simple message.

**Sources:** [Pebble open source (Core Devices)](https://developer.repebble.com/open-source/), [coredevices on GitHub](https://github.com/orgs/coredevices/repositories), [Core Devices app (Google Play)](https://play.google.com/store/apps/details?id=coredevices.coreapp).

---

## 6. Flipper Zero: relay paths (USB serial, infrared, BLE)

**USB serial CLI:** the Flipper exposes a command-line over USB serial. Reach it via serial terminal (`screen`/`minicom` on the `/dev/serial/by-id` path, or a COM port), the browser tool at lab.flipper.net, or qFlipper. Opening the serial port drops you at the CLI. This is the most reliable relay: a host script watches for a trigger and emits OSC into TouchDesigner.

**GPIO:** the CLI has a `gpio` command to set pin mode and read/write state. Pins are 3.3V logic, 5V tolerant on input. Useful for wiring the Flipper to a microcontroller or a simple circuit.

**Infrared:** the IR receiver learns and stores signals for later transmit. A prop path: Flipper IR to a microcontroller IR receiver that then sends serial/OSC.

**BLE:** full BLE peripheral support. The third-party **flipperzero-ble** Python library (async, built on bleak) drives the RPC protocol over BLE, including storage, apps, screen, buttons, Sub-GHz, GPIO, and IR. Good for a wireless trigger without a cable.

**Recommended for the capstone:** USB serial CLI into a Python bridge that emits OSC to TouchDesigner (wired, deterministic, easy to debug), with BLE via flipperzero-ble as the wireless stretch. Hardware detail disagreements exist in sources (IR receiver part number), so trust the official docs and the library README.

**Sources:** [Flipper CLI docs](https://docs.flipper.net/zero/development/cli), [flipperzero-ble (GitHub)](https://github.com/WarrenGreen/flipperzero-ble), [serial CLI access write-up (2025)](https://idefix.net/~koos/newsitem.cgi/1760556560).

---

## 7. Bitfocus Companion with Unreal and TouchDesigner

**TouchDesigner:** a community Companion module controls TouchDesigner over **OSC** (experimental, by Alban Moreon / patopesto, last updated Jan 23 2025). A **generic OSC** module also exists (William Viker).

**Unreal:** no Companion module for Unreal's Remote Control was found. Unreal's Remote Control is HTTP/WebSocket, not OSC, so you either use Companion's generic HTTP actions against the Remote Control endpoints, or route Companion to TouchDesigner over OSC and let TD forward to Unreal.

**Design consequence:** this is a concrete reason to make **TouchDesigner the control hub**. Companion to TD (OSC, supported module) to Unreal (OSC or Remote Control) gives one place to log, debounce, and remap every operator action and prop trigger, which is exactly the hub question the project raised. The reset-take button then lives as one Companion button hitting one TD endpoint.

**Sources:** [companion-module-touch-designer (patopesto)](https://awesome.ecosyste.ms/projects/github.com%2Fpatopesto%2Fcompanion-module-touch-designer), [Companion](https://bitfocus.io/companion).

---

## 8. Enhanced Input with a PlayStation controller and Chaos physics

**Enhanced Input:** the default input system in UE5. Core pieces: Input Mapping Context (IMC), Input Actions (IA), the Enhanced Input Local Player Subsystem, bound in the Player Controller or Pawn. Community guides cover it for 5.5 and general 5.x; gamepad bindings are set in the IMC (most modern gamepads enumerate as a standard gamepad; confirm button mappings on your controller).

**Chaos physics:** UE5's built-in physics engine (replaced PhysX), with rigid-body dynamics, destruction, and ragdoll. Push objects by enabling Simulate Physics on the target and applying force or impulse (`Add Impulse`, `Add Force`) from an input event. **Known gotcha:** a physics body that goes to sleep can stop responding; wake it or disable sleep if pushable objects become unresponsive after settling.

**No single tutorial** covers Enhanced Input plus gamepad plus Chaos push together; combine a push-objects tutorial with an Enhanced Input guide, then test together. This session is low risk given your Unreal background.

**Sources:** [Enhanced Input community tutorial (Epic forums)](https://forums.unrealengine.com/t/community-tutorial-enhanced-input-system-in-unreal-engine-5/2194499), [How to push objects in UE5 (80.lv)](https://80.lv/articles/how-to-create-pushable-objects-in-unreal-engine-5), [physics sleep issue thread](https://forums.unrealengine.com/t/ue-5-0-3-physics-confusion/631525).

---

## 9. Existing TouchDesigner and Unreal toolkits for LED volumes

**Finding:** no open-source toolkit was found that packages the full TD plus Unreal LED-volume loop (device input hub, OSC router, round trip, reset system). The niche this toolkit targets is open. Closest existing pieces to learn from and reuse, not duplicate:

- **TouchEngine-UE and TouchEngine-UE-Samples** (official, GitHub): the sanctioned way to run TD inside UE. Paid-license-gated; study the sample structure for ideas.
- **Off World Live Streaming Toolkit** (Fab, free non-commercial): Spout options for UE5, with a TD component and UE plugin that auto-handle server/client init. Directly reusable for the round trip.
- **Matthew Ragan Post Process Volume** and **CorrelateVisuals/TouchDesigner_Tools**: reusable TD components and effects, good patterns for packaging .tox components.

**Implication for scope:** the toolkit's differentiator is the generic **device-and-operator layer** (audio analysis COMP, device input hub, OSC router, Remote Control receivers, reset system, Companion profiles), not another Spout bridge. Cite Off World Live for the video path rather than rebuilding it.

**Sources:** [Useful GitHub repos for immersive artists (Interactive & Immersive HQ)](https://interactiveimmersive.io/blog/immersive-design-primer/useful-github-repos-for-immersive-artists/), [TouchEngine-UE (GitHub)](https://github.com/TouchDesigner/TouchEngine-UE), [raganmd/touchdesigner_light_volume (GitHub)](https://github.com/raganmd/touchdesigner_light_volume), [CorrelateVisuals/TouchDesigner_Tools (GitHub)](https://github.com/CorrelateVisuals/TouchDesigner_Tools).
