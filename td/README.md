# td/ - TouchDesigner components

Reusable `.tox` components, built across the curriculum. TouchDesigner is the control hub for the whole toolkit.

| Component | Built in | Role |
|---|---|---|
| `audioAnalysis.tox` | S3 | Bands, envelope, beat from live audio; parameterized |
| `oscRouter.tox` | S4, S10 | Namespaced OSC addressing, rate limit, remap table |
| `operatorPanel.tox` | S5, S16 | Operator UI: cues, camera, look toggles |
| `deviceInputHub.tox` | S6 to S8 | Normalizes MIDI, gamepad, serial, and OSC device inputs, debounced |
| `remoteControl.tox` | S14 | Talks to Unreal Remote Control over HTTP/WebSocket |
| `lookLayer.tox` | S18 | Color/comp layer applied to the Unreal feed |

`NAMING.md` holds the component and parameter conventions. Keep every component self-contained (no external path references) so it imports into a blank project and works.

License note: Non-Commercial output is capped at 1280x1280. Build with that in mind.
