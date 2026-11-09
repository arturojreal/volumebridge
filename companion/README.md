# companion/ - Stream Deck and Bitfocus Companion profiles

Exported Companion profiles for operator control.

Routing: Companion to TouchDesigner over OSC (the community TD OSC module), then TD forwards to Unreal. There is no official Companion module for Unreal Remote Control, so TD is the bridge. This keeps one hub for logging, debounce, and remap, and makes the take-reset one button hitting one TD endpoint.

| Item | Built in | Role |
|---|---|---|
| operator profile | S22 | Cue buttons routed through the TD hub |
| reset button | S23 | One-button take reset |

The TD Companion module is community/experimental; a generic OSC module is the fallback. Export profiles here as you build them.
