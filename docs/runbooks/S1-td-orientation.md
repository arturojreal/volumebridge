# S1 runbook - TD orientation and first network

Monday Nov 9. About 3 hours. Machine: MacBook or Windows 4090 (this one is [M/B], so either works, but do it where you will keep building). Follow this top to bottom. Node names match `td/NAMING.md`.

## Goal

A TouchDesigner project that captures live audio and shows a moving waveform and a live spectrum, saved into the repo as `td/host/volumeBridgeHost.toe`.

**Done when:**
- An audio device input is live and its level moves when sound plays.
- A CHOP chain shows a waveform and a spectrum updating in real time.
- The project is saved to `td/host/volumeBridgeHost.toe` and committed.

## Before you start (5 min)

1. Pull the repo so you have the latest plan and the scaffolded `td/host/` folder:
   ```
   git pull
   ```
2. Confirm TouchDesigner 2025 is installed and signed in with a Non-Commercial key. The Non-Commercial banner in the corner is expected.
3. Have an audio source ready: music playing on the machine, or a mic or line input. Note which input you plan to use.

Non-Commercial reminder: TOP (texture) resolution is capped at 1280x1280. Nothing in S1 needs more than that, so it will not bite here. CHOPs (the audio signal chains) are not affected.

## Step 1 - Create and save the project (10 min)

1. Open TouchDesigner. It starts with a default empty network.
2. Save immediately into the repo so autosaves land in the right place:
   - File menu, Save As.
   - Navigate to your local clone, then `td/host/`.
   - Name it `volumeBridgeHost.toe` and save.
3. The `.toe` backup and autosave files are already gitignored, so only the main project gets tracked.

Tip: press Tab or double-click empty network space to open the OP Create dialog. That is how you add every operator below.

## Step 2 - Capture live audio (20 min)

1. Add an **Audio Device In CHOP**. In the OP Create dialog it is under CHOP. Rename it `audioIn` (click the name, or use the Comp/Operator name field).
2. Select the input:
   - On the `audioIn` parameters, open the **Device** menu and pick your input (your mic, line in, or a loopback/aggregate device if you are capturing system audio).
   - On Windows, set **Driver** to ASIO if you have an ASIO device, otherwise DirectSound is fine for S1.
   - Note the **Sample Rate** shown (usually 44100 or 48000). Write it in your log; S2 cares about it.
3. Play sound. The `audioIn` node viewer should show a moving waveform. If it is flat, see Pitfalls below.

## Step 3 - Show a scrolling waveform (15 min)

1. Add a **Trail CHOP**. Rename it `waveTrail`.
2. Wire `audioIn` into `waveTrail` (drag from the output of `audioIn` to the input of `waveTrail`).
3. The `waveTrail` viewer now shows a scrolling history of the waveform. Make its viewer bigger by clicking the node and pressing the viewer-size toggle, or just watch the node's built-in viewer.

## Step 4 - Show a live spectrum (20 min)

1. Add an **Audio Spectrum CHOP**. Rename it `spectrum`.
2. Wire `audioIn` into `spectrum`.
3. The `spectrum` viewer shows magnitude across frequency bins, low frequencies on the left. It should react to music, with bass pushing the left side and highs the right.
4. Optional display as an image: add a **CHOP to TOP**, wire `spectrum` into it, rename it `spectrumTop`. This turns the spectrum into a texture you can show on a panel later. Remember the 1280x1280 cap when you set its resolution.

## Step 5 - A single moving level (15 min)

1. Add an **Analyze CHOP**. Rename it `level`.
2. Wire `audioIn` into `level`.
3. Set the Analyze **Function** to **RMS Power**. The output is now one channel whose value rises and falls with loudness. This is the "level moving" proof for the done checklist and the seed of the envelope follower in S2.
4. Optional: add a **Math CHOP** after `level` to scale it to a comfortable 0 to 1 range, rename it `levelNorm`. Keep the raw `level` too.

## Step 6 - Stretch (optional, if time) 

Make a circle scale with the level:
1. Add a **Circle TOP**, rename it `circle`.
2. Add a **Null CHOP** after `level` (or `levelNorm`), rename it `nullLevel`. A Null is the stable reference point you reference from expressions, so downstream nodes do not break when you rewire.
3. On the `circle` TOP, open the Size parameter and bind it to the level with an expression, for example:
   ```
   0.2 + op('nullLevel')[0]
   ```
4. Play sound. The circle grows and shrinks with loudness.

## Step 7 - Save, log, commit (10 min)

1. Save the project (Ctrl+S or Cmd+S).
2. Add a daily log entry to your tracking log. Note the audio device and sample rate you used, and anything that tripped you up.
3. Commit and push:
   ```
   git add -A
   git commit -m "S1: live audio capture, waveform, spectrum, level in volumeBridgeHost.toe"
   git push
   ```

Note: `.toe` files are binary, so git cannot show a readable diff. That is fine. Commit the file so the project state is shared across machines.

## Pitfalls

- **Flat waveform, no signal:** wrong Device selected, the input is muted at the OS level, or OS mic/recording permission is not granted to TouchDesigner. Check the OS sound settings and the Device menu.
- **Capturing system audio instead of a mic (or vice versa):** on macOS you need a loopback or aggregate device to capture system sound; on Windows, look for a "Stereo Mix" or a virtual loopback input.
- **Sample-rate confusion:** note the rate now. Band analysis in S2 assumes you know it.
- **Reaching for TOP resolution:** the Non-Commercial cap is 1280x1280. If a TOP looks clamped, that is why. CHOP chains are unaffected.
- **Naming drift:** keep the names above (`audioIn`, `waveTrail`, `spectrum`, `level`). S2 and S3 build the `audioAnalysis.tox` component on top of this network, and matching names make that clean.

## What this sets up

- S2 turns `spectrum` and `level` into low, mid, high bands plus an envelope follower.
- S3 wraps that network into `td/audioAnalysis.tox` with Custom Parameters.
- S4 sends the results out as OSC (`/audio/low`, `/audio/env`, and so on) per the address contract in `td/NAMING.md`.

## Docs and tutorials

- Derivative "Learn TouchDesigner" and the Audio Spectrum CHOP and Analyze CHOP wiki pages.
- Elburz, Interactive & Immersive HQ beginner series, for general TD orientation.
