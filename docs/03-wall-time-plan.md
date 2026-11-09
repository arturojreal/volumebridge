# Wall-time plan

The build is laptop-portable by design, so the LED volume, render nodes, and camera tracking are **not required** for any of the 28 sessions. This is deliberate: it removes the biggest schedule risk (shared teaching stage plus a temp assignment window) from the critical path. Wall time becomes a bonus, not a dependency.

## What needs the volume, render nodes, or tracking

| Need | Sessions | Required or optional |
|---|---|---|
| LED volume | S28 (stretch form), optionally a viewing of S18 | **Optional** (showcase only) |
| Stage render nodes | none | not used (locked production cluster) |
| Camera tracking | none | not used (capstone is prop-driven, not tracking-driven) |
| Genlock / timecode | none | not used (laptop output, no genlocked chain) |

Everything else runs on the 4090 laptop with a monitor or a small LED panel as the wall stand-in.

## If you want to showcase on the volume (stretch)

Book one short slot in the final week, ideally for S28 (Fri Dec 18) or a dress-rehearsal pass. Treat it as a demo, not a build session, so nothing you need is blocked if the slot falls through.

Pre-conditions before booking:
1. **Install approval / change-control.** Running a personal laptop as a video source into the stage's routing, or installing anything on stage machines, needs written sign-off. This is a passion project on a shared facility, so get it in writing. See the risk register.
2. **Output path.** The laptop feeds the wall as a video source into existing routing (for example an HDMI or SDI input the stage already accepts), not as software on the render nodes. Confirm the input format the routing expects.
3. **No confidential content.** Nothing stage-specific goes into the public toolkit regardless of where it runs.
4. **Resolution reality.** Non-Commercial TD caps output at 1280x1280, which is fine for a stand-in display but will not fill a large wall at native resolution. For a real wall showcase, plan the output as a windowed region or upgrade the TD license (Educational, USD 300).

## Booking checklist

- Identify a slot that does not collide with classes or cohorts.
- Confirm who must approve a personal-rig video source on the volume.
- Have the full demo working on the laptop first (S27 dress rehearsal), so the stage slot is only about display, not debugging.
- Bring the manual backup triggers (S24) in case a wireless prop misbehaves on the stage network.

## Open input needed from you

Your assignment end date and realistic stage-booking window. The notes suggested a roughly seven-week assignment from late October, which could end before Dec 18. If a volume showcase matters to you, tell me the real window and I will fit the booking around it. If it does not, the laptop demo already satisfies the definition of done.
