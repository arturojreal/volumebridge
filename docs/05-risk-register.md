# Risk register

Each risk has a likelihood, an impact, and a concrete fallback. The build is laptop-portable specifically to retire the biggest ones up front.

| # | Risk | Likelihood | Impact | Fallback |
|---|---|---|---|---|
| 1 | **TD Non-Commercial limits** (1280x1280 cap, no TouchEngine) | Certain | Medium | Pipeline uses OSC + Spout/NDI, which do not need a paid tier. For full-res output or TouchEngine, buy an Educational or Commercial license (Educational is USD 300). Confirmed on Derivative docs. |
| 2 | **No guaranteed LED volume time** (limited or shared stage access) | High | Low (by design) | Demo runs on the laptop rig; the volume is a stretch only. Definition of done is met without wall time. |
| 3 | **Install / change-control not approved** on stage machines | High | Low | Never install on stage machines. If showcasing, feed the wall as a video source into existing routing, with written sign-off. |
| 4 | **UE 5.7.4 Remote Control quirks** (reported Passphrase-header requirement, batch-over-WebSocket issue) | Medium | Medium | Test early (S14). If WebSocket batch fails, use HTTP; if a Passphrase is required, set one. One user report, not confirmed, so verify on your build. |
| 5 | **Spout plugin build mismatch** with UE 5.7 | Medium | Medium | Use the Off World Live plugin build that matches 5.7; if unavailable, fall back to NDI for the round trip (higher latency, same-machine still fine). |
| 6 | **Pebble current SDK HTTP path unverified** | Medium | Medium | Build the phone-side bridge so the watch only sends a simple message; the phone does the HTTP. Confirm the SDK API in coredevices/sdk-docs. Flipper or Stream Deck can stand in as the trigger if Pebble slips. |
| 7 | **Wireless dropouts or prop failure mid-take** | Medium | High (live) | Every prop trigger has a wired path and a manual backup button (S24). The hub logs and debounces so a flaky prop does not double-fire. |
| 8 | **Venue-confidential data leaking into the repo** | Low | High (contractual) | Strict rule: generic repo only, any venue-specific config in a gitignored `private/` overlay. Review before any publish (S25). |
| 9 | **Controller drivers and modes** (a controller needing a vendor driver or a MIDI mode toggle, or a gamepad not detected by Unreal) | Medium | Medium | Test every device in S6 to S8 before it matters; a simple class-compliant pad or encoder controller is the safest fallback if a device fights you. For Unreal gamepads, try Steam Input or DS4Windows. |
| 10 | **Scope creep** (28 sessions, ambitious capstone) | Medium | Medium | Each session has a minimal deliverable plus a stretch. Protect week 6 as buffer; drop stretches before dropping core deliverables. |
| 11 | **Latency stack-up** (device to TD to Unreal to Spout to TD to display) | Medium | Medium | Measure at S17 and S24. Prefer serial/Spout (local) over Wi-Fi/NDI where possible; set a latency budget and cut hops if over it. |
| 12 | **TD learning curve** (new to TouchDesigner) | Medium | Low | Week 1 is fully dedicated to TD fundamentals before any bridging; stretches are optional. |

## The two that actually gate Dec 18

Only risks 4 and 7 can realistically threaten the demo on the laptop. Both are mitigated by testing early (S14 for Remote Control transport, S24 for reliability and manual backups). Everything else is either certain-but-planned-around (1), or downgraded to optional by the laptop-portable decision (2, 3).
