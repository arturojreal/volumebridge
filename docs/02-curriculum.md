# Day-by-day curriculum

6 weeks, 28 weekday sessions of about 3 hours, Monday Nov 9 through Friday Dec 18, 2026. Off Thursday Nov 26 and Friday Nov 27 (Thanksgiving). Build and demo on the Windows RTX 4090 laptop; the LED volume is a stretch showcase only. TouchDesigner is Non-Commercial (1280x1280 cap, no TouchEngine), Unreal is 5.7.4.

**Machine tags:** **[M]** MacBook-friendly, **[W]** Windows 4090, **[B]** both.

Each session lists: deliverable, done checklist, docs and tutorials, pitfalls, stretch, and toolkit contribution. Weekend make-up slots are listed at the end; log them like weekdays.

Learner shape this is tuned to: new to TouchDesigner, strong in Unreal (nDisplay/Switchboard), some Arduino and Python. So week 1 front-loads TD, Unreal weeks move fast, and device work assumes basic flashing.

## Scope and priorities (updated Nov 8)

Core focus, in priority order: TouchDesigner, Unreal Engine, and Stream Deck through Bitfocus Companion, with the Pebble watch as the prop trigger at most. Everything else is nice to have.

Deprioritized to optional: the ESP32 and Arduino button and sensor work (now the firmware appendix) and the Flipper Zero relay (S19). Keep them in the plan as a fallback, but do them only if time allows or a specific need appears.

Device input layer: instead of building new microcontroller firmware, the plan uses MIDI controllers and gamepads as the device input into the TD hub. TouchDesigner has native MIDI In CHOP and DAT support, so this path needs no new parts and no firmware. Week 2 is built around a pad or grid controller, an encoder controller, and gamepads. The firmware sessions moved to an optional appendix.

No new hardware is required for S1 to S5, or for the core TouchDesigner, Unreal, Companion, and Pebble path.

---

## Week 1 (Nov 9 to 13): TouchDesigner foundations and audio-reactive

### S1 - Mon Nov 9 - TD orientation and first network [M/B]
- **Runbook:** step-by-step in `docs/runbooks/S1-td-orientation.md`.
- **Deliverable:** a TD project that captures live audio and shows a waveform and spectrum.
- **Done when:** audio device input is live; a CHOP chain shows level moving; project saved into the repo.
- **Docs/tutorials:** Derivative "Learn TouchDesigner", the official Audio Spectrum and Analyze CHOP wiki pages, Elburz/Interactive & Immersive HQ beginner series.
- **Pitfalls:** picking the wrong audio device; sample-rate mismatches; forgetting that Non-Commercial caps TOP resolution at 1280x1280.
- **Stretch:** add a simple audio-driven TOP (a circle that scales with level).
- **Toolkit:** repo skeleton, `td/` folder, a NAMING.md with component and parameter conventions.

### S2 - Tue Nov 10 - Audio analysis deep dive [M/B]
- **Deliverable:** low/mid/high band energy plus an envelope follower, as clean CHOP channels.
- **Done when:** three band channels and one envelope channel respond correctly to music and to voice.
- **Docs/tutorials:** Audio Spectrum CHOP, Analyze CHOP, Filter and Lag CHOP wiki pages.
- **Pitfalls:** no smoothing (jittery output); band crossovers overlapping; mono vs stereo confusion.
- **Stretch:** add beat detection (a trigger on envelope threshold).
- **Toolkit:** `audioAnalysis` network, pre-component.

### S3 - Wed Nov 11 - Componentize the audio analysis [M/B]
- **Deliverable:** `audioAnalysis.tox` with Custom Parameters (gain, smoothing, band crossovers) and named outputs.
- **Done when:** the .tox drops into a blank project and works with no external references; parameters change behavior live.
- **Docs/tutorials:** Derivative "Components", "Custom Parameters", "Promoting Parameters".
- **Pitfalls:** hard-coded paths that break on import; unexposed parameters; no default values.
- **Stretch:** add a built-in preview panel on the COMP.
- **Toolkit:** first real reusable component, `td/audioAnalysis.tox` v0.1.

### S4 - Thu Nov 12 - OSC out from TD [M/B]
- **Deliverable:** TD sends band and envelope values as namespaced OSC (for example `/audio/low`, `/audio/env`).
- **Done when:** an OSC monitor shows the addresses and values updating at a steady rate.
- **Docs/tutorials:** OSC Out CHOP/DAT wiki; Protokol or an OSC monitor app.
- **Pitfalls:** flooding the network with too-high send rates; address typos; wrong port.
- **Stretch:** add a send-rate limiter parameter.
- **Toolkit:** `td/oscRouter.tox` v0.1 (address namespacing, rate limit).

### S5 - Fri Nov 13 - Control panel basics and weekly review [M/B]
- **Deliverable:** a small operator panel (sliders, toggles) driving the audio COMP, plus the week-1 review.
- **Done when:** panel controls change audio-analysis behavior live; daily logs and the toolkit list are current.
- **Docs/tutorials:** Derivative "Widgets", "Panel COMPs", Container COMP layout.
- **Pitfalls:** binding widgets to the wrong parameters; layout breaking at different window sizes.
- **Stretch:** save/recall of panel presets.
- **Toolkit:** `td/operatorPanel.tox` v0.1; weekly review note in the tracking log.

---

## Week 2 (Nov 16 to 20): MIDI and gamepad input, and the OSC bridge to Unreal

> Scope note: Week 2 uses MIDI controllers and gamepads for device input. The original ESP32/Arduino firmware sessions moved to the Optional firmware appendix at the end. The OSC bridge to Unreal in S9 and S10 stays core.

### S6 - Mon Nov 16 - MIDI into TouchDesigner [B]
- **Deliverable:** an encoder controller and a pad or grid controller both arrive in TD, with encoders and pads mapped to named channels.
- **Done when:** turning an encoder moves a named channel smoothly and pressing a pad fires a clean trigger; a mapping table (device, control, channel name) lives in a Table DAT.
- **Docs/tutorials:** Derivative MIDI In CHOP, MIDI In DAT, and MIDI Device Mapper wiki pages; your controller's configuration-utility manual.
- **Pitfalls:** encoders in relative mode (values jump or wrap); two apps grabbing the same MIDI port (close any DAW); MIDI channel mismatches.
- **Stretch:** drive the S5 operator panel from the encoders.
- **Toolkit:** `deviceInputHub` MIDI input stage and its mapping table.

### S7 - Tue Nov 17 - Two-way MIDI and bonus controllers [B]
- **Deliverable:** a grid-controller cue grid that lights pads to show cue state, plus any extra pad or keyboard controllers feeding the hub.
- **Done when:** pressing a grid pad fires a cue and lights it until the cue ends. Extra controllers are bonus: map them only if they have a job in the scene.
- **Docs/tutorials:** Derivative MIDI Out CHOP and DAT; your grid controller's programmer's reference (note-to-color values); the vendor MIDI-mode utility docs.
- **Pitfalls:** a controller still in its native mode instead of MIDI mode; feedback loops when MIDI out echoes back in; wrong color velocity values on the grid.
- **Stretch:** a controller's line out into the audio input, so the same device both plays and controls.
- **Toolkit:** grid cue-grid feedback logic; device ids for your controllers added to `td/NAMING.md`.

### S8 - Wed Nov 18 - Gamepads and the device hub [B]
- **Deliverable:** gamepads read in TD, and `deviceInputHub.tox` v0.1 normalizing MIDI and gamepad input into the `/device` OSC namespace.
- **Done when:** sticks and buttons from at least two controllers show as named channels; the hub debounces triggers and outputs normalized `/device/<id>/<control>` values; Unreal gamepad detection is checked on Windows so S12 is not a surprise.
- **Docs/tutorials:** Derivative Joystick CHOP; Epic Enhanced Input docs; Steam Input or DS4Windows docs if Unreal does not see the controllers.
- **Pitfalls:** some controllers not detected as a standard gamepad; controller indexes changing between sessions (map by name, not index); drift on worn sticks (add deadzones).
- **Stretch:** TD MIDI out sequencing an external MIDI instrument (needs a USB MIDI interface with 5-pin DIN out) with its audio feeding `audioAnalysis`.
- **Toolkit:** `td/deviceInputHub.tox` v0.1 (MIDI and gamepad inputs, debounced, normalized).

### S9 - Thu Nov 19 - Unreal OSC receiver [W]
- **Deliverable:** Unreal receives TD OSC and moves an actor and a material parameter from the audio bands.
- **Done when:** a cube scales/rotates and a material value tracks audio, live from TD.
- **Docs/tutorials:** Epic OSC plugin overview; `On OSC Message Received` Blueprint event.
- **Pitfalls:** OSC plugin not enabled; server not listening on the expected port; address-pattern mismatch.
- **Stretch:** map three different addresses to three different actors.
- **Toolkit:** `unreal/` template project with an OSC receiver Blueprint and a documented address map.

### S10 - Fri Nov 20 - Harden the bridge and weekly review [W]
- **Deliverable:** a documented OSC contract (addresses, types, ranges) and a remap table in TD.
- **Done when:** missing or extra OSC args do not crash either side; a logging view shows traffic; review done.
- **Docs/tutorials:** Epic OSC server allowlist and address binding; TD DAT-based remap patterns.
- **Pitfalls:** silent failures when a type is wrong; no logging; addresses drifting between TD and Unreal.
- **Stretch:** a self-test mode that fires every address once.
- **Toolkit:** `oscRouter.tox` v0.2; `docs` OSC address spec; weekly review note.

---

## Week 3 (Nov 23 to 25): Unreal scene, gamepad physics, on-screen controllers (short week)

### Before S11 - Scene brief (any weekday slack or a weekend, about 1 hour) [M]
- **Deliverable:** pick a scene concept and write a one-page `docs/scene-brief.md`: the story beat, what the prop press reveals, what the camera does, what the audio drives, which physics objects the gamepad pushes, and which devices get a job.
- **Done when:** every device you plan to use has one clear job in the scene, and S11 has a concrete build list.
- **Why now:** the S11 scene build and the device mappings in S6 to S8 should serve one idea instead of being wired up in the abstract.

### S11 - Mon Nov 23 - Scene build [W]
- **Deliverable:** a small lit scene using Fab/Quixel assets plus one original asset, staged for a reveal.
- **Done when:** scene runs at a comfortable frame rate on the 4090; a clear spot for a "screen appears" moment.
- **Docs/tutorials:** Fab, Quixel Bridge, Lumen basics; Epic "Lighting the Environment".
- **Pitfalls:** overscoping the scene; heavy assets tanking frame rate; forgetting this must survive a reset.
- **Stretch:** a simple Sequencer camera move.
- **Toolkit:** `unreal` template scene (kept generic, no stage-specific content).

### S12 - Tue Nov 24 - Gamepad and Chaos physics [W]
- **Deliverable:** a gamepad pushes Chaos physics props with impulse via Enhanced Input.
- **Done when:** a button or stick applies force to simulating bodies; objects behave believably.
- **Docs/tutorials:** Enhanced Input community tutorial; "How to push objects in UE5" (80.lv); Chaos overview.
- **Pitfalls:** physics bodies sleeping and going unresponsive; input context not added; impulse scale off.
- **Stretch:** a "nudge" vs "launch" based on hold duration.
- **Toolkit:** a reusable Input Mapping Context and a `Pushable` Blueprint.

### S13 - Wed Nov 25 - On-screen controllers and a Remote Control preset [W]
- **Deliverable:** a UMG or Editor Utility Widget plus a Remote Control Preset exposing the reveal, camera, and physics-reset parameters.
- **Done when:** the web interface loads the preset; each exposed control works from inside the editor.
- **Docs/tutorials:** Epic "Remote Control for UE", "Remote Control Web Interface", UMG quick start.
- **Pitfalls:** exposing the wrong properties; preset not registering; port conflicts.
- **Stretch:** group controls into a tidy panel layout.
- **Toolkit:** a documented Remote Control Preset (the trigger surface the whole capstone uses).

---

## Week 4 (Nov 30 to Dec 4): Remote Control, web panels, TD control panels, video round trip

### S14 - Mon Nov 30 - Remote Control over HTTP and WebSocket [W]
- **Deliverable:** fire a scene change from a REST client and from TouchDesigner over the Remote Control API.
- **Done when:** an external request changes a property live; the 5.7.4 Passphrase-header behavior is confirmed or ruled out on your build.
- **Docs/tutorials:** Epic WebSocket reference; the 5.7.4 batch-over-WebSocket forum thread (treat as a caution, not gospel).
- **Pitfalls:** 5.7.4 may require a Passphrase header; `/remote/batch` may not work over WebSocket; server only runs when a project is open.
- **Stretch:** a WebSocket client in TD that subscribes to preset events.
- **Toolkit:** `td/remoteControl.tox` and a Remote Control client cheatsheet.

### S15 - Tue Dec 1 - Web control panel [W]
- **Deliverable:** a Remote Control Web Interface panel with reveal and reset buttons, reachable from a phone on the LAN.
- **Done when:** a phone browser on the same network triggers the scene; buttons are labeled and safe.
- **Docs/tutorials:** Epic "Remote Control Web Interface".
- **Pitfalls:** exposing the server beyond the LAN (do not); firewall blocking :30010; mixed-content issues.
- **Stretch:** a minimal custom web page hitting the HTTP endpoints directly.
- **Toolkit:** exported web panel config.

### S16 - Wed Dec 2 - TD control panels for camera and cues [B]
- **Deliverable:** Widget COMPs that send OSC or Remote Control calls to move an Unreal camera and fire a Sequencer cue.
- **Done when:** a TD panel drives a camera move and starts a cue reliably.
- **Docs/tutorials:** TD Widgets; Epic Sequencer scripting; the OSC address spec from S10.
- **Pitfalls:** cue timing races; camera control fighting Sequencer; panel state drifting from engine state.
- **Stretch:** two-way state so the panel reflects what Unreal is doing.
- **Toolkit:** `operatorPanel.tox` v0.2 (camera and cue controls).

### S17 - Thu Dec 3 - Spout round trip out of Unreal [W]
- **Deliverable:** an Unreal frame arrives in TouchDesigner via the Off World Live Spout plugin.
- **Done when:** TD shows the live Unreal render through a Spout In TOP with low latency.
- **Docs/tutorials:** Off World Live Spout plugin docs; TD Spout In TOP wiki.
- **Pitfalls:** Spout is same-machine only; plugin build must match UE 5.7; color space surprises.
- **Stretch:** measure round-trip latency and note it.
- **Toolkit:** Unreal Spout sender setup and a TD Spout In node.

### S18 - Fri Dec 4 - Look layer, output, and weekly review [W]
- **Deliverable:** a TD color/comp layer on the Unreal feed, output to a second display as the wall stand-in.
- **Done when:** the full round trip runs (Unreal to TD to display) with a look applied; review done.
- **Docs/tutorials:** TD compositing TOPs; Derivative color-correction examples.
- **Pitfalls:** the 1280x1280 Non-Commercial cap limits output resolution; double color correction; added latency.
- **Stretch:** an A/B toggle for the look layer.
- **Toolkit:** `td/lookLayer.tox` v0.1; weekly review note.

---

## Week 5 (Dec 7 to 11): Capstone prop-driven storytelling and operator controls

### S19 - Mon Dec 7 - Flipper Zero relay [B] (optional)
- **Deliverable:** a Flipper button press fires an OSC event into the TD device hub via a Python serial bridge.
- **Done when:** pressing on the Flipper reliably produces one debounced OSC event in TD.
- **Docs/tutorials:** Flipper CLI docs; pyserial; the device-hub address spec.
- **Pitfalls:** serial port permissions; double-fire without debounce; CLI prompt parsing.
- **Stretch:** BLE path via the flipperzero-ble Python library (wireless).
- **Toolkit:** `props/flipper_bridge.py`.

### S20 - Tue Dec 8 - Pebble watch relay [B]
- **Deliverable:** a watch press reaches Unreal Remote Control (or TD) through the phone companion.
- **Done when:** a watch button triggers a preset end to end; a wired or manual backup exists.
- **Docs/tutorials:** developer.repebble.com; coredevices/sdk-docs; the Remote Control cheatsheet from S14.
- **Pitfalls:** current SDK HTTP path is unverified (build the phone-side bridge so the watch sends a simple message); BLE pairing; latency.
- **Stretch:** two-way feedback (watch shows "armed").
- **Toolkit:** `props/pebble/` app and bridge notes.

### S21 - Wed Dec 9 - Capstone wiring [W]
- **Deliverable:** watch or Flipper trigger to the TD hub (log, debounce, remap) to Unreal: a screen reveal and a camera move.
- **Done when:** the storytelling beat runs end to end on the laptop: prop fires, screen appears, camera moves.
- **Docs/tutorials:** everything from weeks 2 to 4; the hub pattern from brief 7.
- **Pitfalls:** trigger races; no single source of truth for state; hard-to-debug cross-app timing.
- **Stretch:** a second prop that does a different beat.
- **Toolkit:** the capstone example project (generic story, not stage-specific).

### S22 - Thu Dec 10 - Stream Deck and Companion [B]
- **Deliverable:** Companion to TD (OSC module) to Unreal, with physical buttons for cues.
- **Done when:** Stream Deck buttons fire cues through the TD hub reliably.
- **Docs/tutorials:** Bitfocus Companion docs; the community TD OSC module (patopesto); generic OSC module.
- **Pitfalls:** the TD Companion module is experimental; no official Unreal module (route via TD); button-state feedback.
- **Stretch:** button color feedback reflecting engine state.
- **Toolkit:** Companion profile export in `companion/`.

### S23 - Fri Dec 11 - Reset-take system and weekly review [W]
- **Deliverable:** one button restores physics objects, camera, Sequencer playhead, and scene state.
- **Done when:** after a messy take, the reset returns everything to start cleanly and repeatably; review done.
- **Docs/tutorials:** Epic Blueprint state-save patterns; Sequencer playhead control.
- **Pitfalls:** physics not fully re-settling; partial resets; order-of-operations bugs.
- **Stretch:** a soft reset (keep look settings) vs a hard reset.
- **Toolkit:** the `reset` system (Unreal Blueprint plus a TD/Companion button); weekly review note.

---

## Week 6 (Dec 14 to 18): Packaging, docs, reliability, demo

### S24 - Mon Dec 14 - Reliability hardening [B]
- **Deliverable:** a latency note, Wi-Fi dropout handling, and manual backup triggers for when a prop fails mid-take.
- **Done when:** pulling the Wi-Fi or a prop does not kill the show; a manual trigger covers every prop event.
- **Docs/tutorials:** your own latency measurements; OSC/serial reconnect patterns.
- **Pitfalls:** assuming the wireless path always works; no manual fallback; silent reconnection failures.
- **Stretch:** a health dashboard in TD (which inputs are alive).
- **Toolkit:** a reliability checklist and a manual-trigger panel.

### S25 - Tue Dec 15 - Packaging and versioning [M/B]
- **Deliverable:** a clean repo tree, consistent component naming, stage-specific config isolated in the gitignored `private/` overlay, and a v0.1 tag plan.
- **Done when:** a fresh clone has everything generic and nothing stage-specific; licensing chosen (MIT suggested).
- **Docs/tutorials:** `04-toolkit-architecture.md`; choosealicense.com.
- **Pitfalls:** stage-specific or confidential content leaking into the repo; broken relative paths; missing defaults.
- **Stretch:** a one-command setup check script.
- **Toolkit:** finalized structure and version scheme.

### S26 - Wed Dec 16 - Documentation [M]
- **Deliverable:** README, setup guide, one example walk-through, and the "what to change for other volumes" write-up.
- **Done when:** someone else could set up the toolkit from the docs alone.
- **Docs/tutorials:** this repo's docs; your own daily logs as raw material.
- **Pitfalls:** docs that assume your exact machine; missing version pins; no troubleshooting section.
- **Stretch:** a short demo video or GIF.
- **Toolkit:** README plus setup guide complete.

### S27 - Thu Dec 17 - Dress rehearsal [W]
- **Deliverable:** a full end-to-end run on the laptop rig, timed, with a cleared punch list.
- **Done when:** the demo runs start to finish twice without intervention.
- **Docs/tutorials:** your demo runbook.
- **Pitfalls:** first-run-only bugs; state not resetting between runs; forgetting the backup triggers.
- **Stretch:** rehearse the failure path (prop dies, operator recovers).
- **Toolkit:** the demo runbook.

### S28 - Fri Dec 18 - Demo, publish, retro [W] (volume = stretch)
- **Deliverable:** run the demo (laptop, or the volume if booked), publish toolkit v0.1, write the short retro.
- **Done when:** the definition-of-done items are met (see below); v0.1 is published; the retro is written.
- **Docs/tutorials:** `05-risk-register.md` for the stage-time fallback.
- **Pitfalls:** depending on wall time that did not materialize; last-minute version drift.
- **Stretch:** the demo actually runs on the LED volume with permission and a booked slot.
- **Toolkit:** v0.1 published with README, setup guide, and the capstone example.

---

## Definition of done (Dec 18), mapped to sessions

- Prop trigger fires a screen reveal and a camera move: S20 to S21 (Pebble is the primary prop; Flipper S19 is optional).
- An audio-reactive or TD look layer in the pipeline: S2 to S4, S18.
- A Stream Deck reset between takes: S22 to S23.
- Toolkit v0.1 published with README, setup guide, one example: S25 to S28.
- A short write-up of what worked here and what to change for other volumes: S26, S28.

Note: the live demo runs on the laptop rig by default. "On the volume" is the stretch form of S28 and depends on booked wall time and install approval.

## Weekend make-up slots

If a weekday slips, use Saturday or Sunday of the same week. Likely candidates to pull forward or make up: the device sessions (S6 to S8) and the capstone wiring (S21). Log every weekend session in the tracking log like a weekday so slippage stays visible.

## Optional firmware appendix

The original Week 2 sessions, kept as a fallback if a custom sensor or prop ever needs firmware. Not scheduled. Use a weekend slot if you take one on.

### F1 - ESP32/Arduino to TD over serial [B]
- **Deliverable:** a board reads a button and an analog sensor; TD shows the values via a Serial DAT.
- **Done when:** values update live in TD; a documented serial format (comma-separated, one line per frame).
- **Docs/tutorials:** Interaction Station "TouchDesigner & Arduino", MIT Fab serial notes, Serial DAT wiki.
- **Pitfalls:** wrong COM port or baud; missing USB driver (CP210x/CH340); line-ending parsing.
- **Stretch:** add a second sensor and parse multiple values.
- **Toolkit:** `firmware/basic_io/` sketch and a TD serial-parse snippet.

### F2 - Two-way serial [B]
- **Deliverable:** TD sends values back to the board to drive an LED or a servo.
- **Done when:** a TD slider moves a servo or dims an LED; both directions run at once without blocking.
- **Docs/tutorials:** Serial DAT send examples; Arduino `Serial.read` parsing patterns.
- **Pitfalls:** serial buffer overrun; blocking delays in the sketch; echo loops.
- **Stretch:** a simple handshake so the board announces itself.
- **Toolkit:** firmware two-way example; `deviceInputHub` input stub in TD.

### F3 - ESP32 OSC over Wi-Fi [B]
- **Deliverable:** ESP32 sends and receives OSC over UDP to and from TD.
- **Done when:** wireless both-direction messaging works; wired serial still available as fallback.
- **Docs/tutorials:** Arduino OSC library; the touchdesigner-pca9685-servo-control project (two-way OSC pattern); ESP32 UDP examples.
- **Pitfalls:** Wi-Fi latency and dropouts; UDP packet loss; firewall blocking the port.
- **Stretch:** auto-reconnect on Wi-Fi drop.
- **Toolkit:** serial and OSC input stages added to `td/deviceInputHub.tox`.
