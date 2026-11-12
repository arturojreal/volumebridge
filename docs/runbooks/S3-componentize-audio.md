# S3 runbook - Componentize the audio analysis

About 3 hours. Machine: [M/B], either works. You will copy the S2 network into a component called `audioAnalysis`, give it Custom Parameters so you can tune it without opening it, save it as `td/audioAnalysis.tox`, and prove it works in a blank project. Node names match your S2 network and `td/NAMING.md`.

UI labels marked **[verify]** are from memory. If one reads differently in your build, use the closest match.

## Goal

A self-contained `audioAnalysis` component: audio goes into its one input, and `low`, `mid`, `high`, `env`, and `beat` come out of its outputs. Its Custom Parameters control gain, crossovers, per-band scaling, smoothing, and beat threshold.

**Done when:**
- `td/audioAnalysis.tox` exists and is committed.
- Dropped into a brand-new project with an Audio Device In CHOP wired to it, it outputs all five channels with no errors.
- Changing **Gain**, **Lowcrossover**, **Smoothing**, and **Beatthreshold** visibly changes the outputs.

## Why copy instead of moving

You will copy the S2 nodes into the component and leave the originals in place until the component is proven. If anything goes wrong, your working S2 network is untouched. Step 11 deletes the originals at the end.

## Before you start (10 min)

1. `git pull` in your clone, then open `td/host/volumeBridgeHost.toe`.
2. Save once with Ctrl+S, then commit, so you have a clean restore point before restructuring.
3. Re-measure your peaks at the volume you will actually work at (from the S2 log, the last values were from a quiet test). Play the loud part of your test track for at least 20 seconds and read these four nodes:
    - `lowSmoothAnalyze`
    - `midSmoothAnalyze`
    - `highSmoothAnalyze`
    - `envAnalyze`
4. Multiply each by `0.9` and write the four results down. Worked example: `lowSmoothAnalyze` reads `0.82`, so `0.82 x 0.9 = 0.74`. These become the default scale values in step 6.

Check: you have four numbers written down, labeled low, mid, high, and env.

## Step 1 - Create the empty component (5 min)

1. In `/project1`, press Tab and add a **Base COMP**. Rename it `audioAnalysis`.
2. Place it to the right of `audioIn`, with some empty space around it.

Check: an empty `audioAnalysis` node sits in `/project1`.

## Step 2 - Copy the S2 nodes in (15 min)

1. In `/project1`, select exactly these nodes. Ctrl+click adds one at a time; you can also drag a selection box and then Ctrl+click to remove extras.
    - Audio path: `mono`, `level`
    - Filters: `lowFilter`, `midHigh`, `midFilter`, `highFilter`
    - Loudness: `lowLevel`, `midLevel`, `highLevel`
    - Smoothing: `lowSmooth`, `midSmooth`, `highSmooth`, `envSmooth`
    - Scaling: `lowNorm`, `midNorm`, `highNorm`, `envNorm`
    - Clamps: `lowLimit`, `midLimit`, `highLimit`
    - Names: `lowName`, `midName`, `highName`, `envName`
    - Outputs: `bandMerge`, `out_bands`, `out_env`
    - Beat: `beatTrigger`, `beatName`, `out_beat`
    - Peak meters (kept inside as tuning helpers): `lowSmoothTrail`, `lowSmoothAnalyze`, `midSmoothTrail`, `midSmoothAnalyze`, `highSmoothTrail`, `highSmoothAnalyze`, `envTrail`, `envAnalyze`
2. Do **not** select: `audioIn`, `waveTrail`, `spectrum`, `spectrumTop`, `levelNorm`, `nullLevel`, `circle`. Those stay in the host.
3. Press Ctrl+C.
4. Double-click `audioAnalysis` to go inside it (or select it and press I). The network is empty.
5. Press Ctrl+V and click in empty space to place the copies.

Check: inside `audioAnalysis` you see all the copied nodes, still wired to each other exactly as they were in `/project1`. `mono` and `level` show no input, which is expected; step 3 fixes that.

## Step 3 - Give the component an input and a gain stage (15 min)

1. Inside `audioAnalysis`, add an **In CHOP**. Rename it `in_audio`. Place it at the far left.
2. Add a **Math CHOP**. Rename it `inputGain`. Wire `in_audio` into `inputGain`.
3. Wire `inputGain` into `mono`.
4. Wire `inputGain` into `level` as well. One output can feed several nodes.
5. Leave `inputGain`'s parameters at their defaults for now. Step 7 connects its **Multiply** to the Gain parameter.

Check: `mono` and `level` both show `inputGain` as their input. Nothing shows live values yet, because nothing feeds the component.

## Step 4 - Give the component outputs (15 min)

An Out CHOP inside a component becomes an output connector on the component. Your three Null nodes already use the names the outputs should have, so rename them first.

1. Rename the Null `out_bands` to `null_bands`.
2. Rename the Null `out_env` to `null_env`.
3. Rename the Null `out_beat` to `null_beat`.
4. Add an **Out CHOP** named `out_bands`. Wire `null_bands` into it.
5. Add an **Out CHOP** named `out_env`. Wire `null_env` into it.
6. Add an **Out CHOP** named `out_beat`. Wire `null_beat` into it.
7. Arrange the three Out CHOPs top to bottom in that order: `out_bands`, `out_env`, `out_beat`. Their vertical order sets the order of the component's output connectors.

Check: the three Out CHOPs exist, each wired from its matching `null_...` node.

## Step 5 - Connect it in the host (10 min)

1. Press U to go back up to `/project1`.
2. `audioAnalysis` now shows one input connector on its left and three output connectors on its right.
3. Wire `audioIn` into `audioAnalysis`'s input.
4. Add three **Null CHOPs** to the right of `audioAnalysis`, named `aa_bands`, `aa_env`, `aa_beat`. Wire the first output into `aa_bands`, the second into `aa_env`, the third into `aa_beat`.
5. Play music.

Check: `aa_bands` shows three channels named `low`, `mid`, `high` moving with the music; `aa_env` shows `env`; `aa_beat` pulses `beat` on kicks. These match your original S2 outputs.

## Step 6 - Create the Custom Parameters (30 min)

Custom Parameters are the knobs on the outside of the component. TouchDesigner requires each parameter name to start with a capital letter followed by lowercase letters or digits, which is why the names below look like `Lowcrossover`.

1. Right-click `audioAnalysis` and choose **Customize Component...** **[verify]**. The Component Editor opens.
2. Create a page called **Settings**: type `Settings` in the page name field and click **Add Page** **[verify]**.
3. With the **Settings** page selected, add each parameter below. For each one: choose the type in the parameter type menu, type the name, click **Add Par** **[verify]**, then select it in the list and fill in its Label, Default, Min, and Max.

| Name | Type | Label | Default | Min | Max | What it controls |
|---|---|---|---|---|---|---|
| `Gain` | Float | Gain | `1` | `0` | `8` | Input level before analysis; your volume knob |
| `Lowcrossover` | Float | Low crossover (Hz) | `250` | `40` | `1000` | Top of the low band, bottom of the mid band |
| `Highcrossover` | Float | High crossover (Hz) | `4000` | `1000` | `12000` | Top of the mid band, bottom of the high band |
| `Lowscale` | Float | Low scale | your low value from Before you start | `0.001` | `2` | Level that maps to 1 for low |
| `Midscale` | Float | Mid scale | your mid value | `0.001` | `2` | Level that maps to 1 for mid |
| `Highscale` | Float | High scale | your high value | `0.001` | `2` | Level that maps to 1 for high |
| `Envscale` | Float | Envelope scale | your env value | `0.001` | `2` | Level that maps to 1 for env |
| `Smoothing` | Float | Smoothing | `1` | `0.1` | `4` | Multiplies every fall time; higher is smoother |
| `Beatthreshold` | Float | Beat threshold | `0.6` | `0` | `1` | How loud the low band must get to fire a beat |

4. Add a second page called **About**. On it, add a **Str** (string) parameter named `Version`, Label `Version`, Default `0.1.0`.
5. Close the Component Editor.

Check: click `audioAnalysis` and press P. Its parameter window shows a **Settings** page with the nine parameters at their defaults and an **About** page with Version `0.1.0`. Changing them does nothing yet; step 7 wires them up.

## Step 7 - Wire the parameters to the nodes (35 min)

Each node inside reads its value from the component's parameters through an expression. `parent()` means "the component I am inside", so `parent().par.Gain` reads the Gain parameter of `audioAnalysis`.

How to enter an expression: on the parameter row, click the small mode button at the left edge of the value field until the field turns blue (Expression mode), then type the expression and press Enter. You did this once in S1, on the circle's Radius.

Go inside `audioAnalysis` (double-click it), then set each of these:

**Gain**
1. `inputGain` (Math CHOP), **Mult-Add** page, **Multiply** field: `parent().par.Gain`

**Crossovers** (on each Audio Filter CHOP, the **Filter Cutoff (Hz)** field)
2. `lowFilter`: `parent().par.Lowcrossover`
3. `midHigh`: `parent().par.Lowcrossover`
4. `midFilter`: `parent().par.Highcrossover`
5. `highFilter`: `parent().par.Highcrossover`

**Scaling** (on each Math CHOP, **Range** page, **From Range**, the second field only)
6. `lowNorm`: `parent().par.Lowscale`
7. `midNorm`: `parent().par.Midscale`
8. `highNorm`: `parent().par.Highscale`
9. `envNorm`: `parent().par.Envscale`

**Smoothing** (on each Lag CHOP, **Lag** parameter, the second field only; leave the first field as a number)
10. `lowSmooth`: `0.25 * parent().par.Smoothing`
11. `midSmooth`: `0.15 * parent().par.Smoothing`
12. `highSmooth`: `0.1 * parent().par.Smoothing`
13. `envSmooth`: `0.4 * parent().par.Smoothing`

**Beat**
14. `beatTrigger` (Trigger CHOP), **Trigger Threshold** field: `parent().par.Beatthreshold`

Check: press U to go back up. With music playing, change each parameter on `audioAnalysis` and watch the `aa_...` Nulls:
- **Gain** from `1` to `2`: all bands and `env` roughly double, then clamp at 1.
- **Lowcrossover** from `250` to `120`: `low` drops noticeably and `mid` picks up.
- **Smoothing** from `1` to `3`: every channel falls more slowly after loud moments.
- **Beatthreshold** from `0.6` to `0.9`: `beat` fires less often.

Set them all back to their defaults when done (right-click a parameter and choose **Reset to Default** **[verify]**, or retype the value).

## Step 8 - Add a README inside (10 min)

1. Inside `audioAnalysis`, add a **Text DAT**. Rename it `README`.
2. Click it, then open its editor (right-click and choose **Edit Contents...** **[verify]**, or use the viewer) and type:

```
audioAnalysis v0.1.0
Input: one audio CHOP (any channel count; it is averaged to mono).
Outputs: 1 out_bands (low, mid, high), 2 out_env (env), 3 out_beat (beat). All 0 to 1.
Settings: Gain (input level), Lowcrossover / Highcrossover (Hz), Low/Mid/High/Envscale
(level that maps to 1), Smoothing (fall-time multiplier), Beatthreshold.
Calibrate: play loud audio, read the ...Analyze peak meters inside, set each scale to 0.9 x peak.
OSC (S4): /audio/low, /audio/mid, /audio/high, /audio/env, /audio/beat
```

Check: the `README` DAT shows the text.

## Step 9 - Save the component as a .tox (10 min)

1. Press U to go back up to `/project1`.
2. Right-click `audioAnalysis` and choose **Save Component .tox...** **[verify]**.
3. Navigate to your clone's `td/` folder (one level above `td/host/`). Name the file `audioAnalysis.tox` and save.
4. Save the host project with Ctrl+S.

Check: in your file browser, `td/audioAnalysis.tox` exists next to `NAMING.md`.

## Step 10 - Prove it works in a blank project (25 min)

This is the real test of "self-contained": nothing in the component may depend on your host project.

1. Make sure the host is saved, then choose **File > New**. A blank project opens.
2. Drag `td/audioAnalysis.tox` from your file browser into the empty network.
3. Add an **Audio Device In CHOP** and wire it into the component's input.
4. Add three **Null CHOPs** and wire one to each output.
5. Play music.
6. Click the component and look for a red error badge on it or inside it. Double-click into it and check that no node shows an error.

Check, all four must be true:
- The three Nulls show `low`, `mid`, `high`; `env`; and `beat`, all moving.
- No node inside shows an error.
- Changing **Gain** changes the outputs.
- The **About** page shows Version `0.1.0`.

7. Close this test project without saving it, and reopen `td/host/volumeBridgeHost.toe`.

If something errors: the usual cause is an expression or wire that still points at a node outside the component. Go to the node with the error, find the field that names something outside (for example `op('/project1/...')`), and change it to a node inside or to `parent().par...`. Then save the .tox again (step 9) and repeat step 10.

## Step 11 - Remove the originals from the host (10 min)

Now that the component works, delete the loose S2 copies in `/project1` so there is one source of truth.

1. In `/project1`, select and delete these original nodes: `mono`, `lowFilter`, `midHigh`, `midFilter`, `highFilter`, `lowLevel`, `midLevel`, `highLevel`, `lowSmooth`, `midSmooth`, `highSmooth`, `envSmooth`, `lowNorm`, `midNorm`, `highNorm`, `envNorm`, `lowLimit`, `midLimit`, `highLimit`, `lowName`, `midName`, `highName`, `envName`, `bandMerge`, `out_bands`, `out_env`, `beatTrigger`, `beatName`, `out_beat`, and the eight peak-meter nodes.
2. Keep `level` in the host. `levelNorm`, `nullLevel`, and `circle` from S1 still read from it.
3. Keep `audioIn`, `audioAnalysis`, and the three `aa_...` Nulls.
4. Save with Ctrl+S.

Check: `/project1` contains `audioIn`, `audioAnalysis`, `aa_bands`, `aa_env`, `aa_beat`, plus the S1 nodes (`waveTrail`, `spectrum`, `spectrumTop`, `level`, `levelNorm`, `nullLevel`, `circle`). No node shows an error, and the circle still pulses.

## Step 12 - Stretch: a preview panel (optional, 30 min)

1. Inside `audioAnalysis`, add a **CHOP to TOP** named `previewTop` that references `null_bands` through its **CHOP** parameter (drag `null_bands` onto the field).
2. Add a **Fit TOP** after it set to `256` by `64`.
3. On `audioAnalysis`, open the **Common** page **[verify]** and set its node viewer to show that TOP, so the component's own tile shows the bands.

Check: the `audioAnalysis` tile in `/project1` shows a live strip of the three bands.

## Step 13 - Save, log, commit (10 min)

1. Save with Ctrl+S. If you changed anything inside the component after step 9, save the .tox again (step 9).
2. Add a daily log entry to your tracking log: note your four scale defaults, whether the preview-panel stretch was done, and anything that tripped you up. Add `audioAnalysis.tox` v0.1.0 to the running toolkit list.
3. Commit and push:
   ```
   git add -A
   git commit -m "S3: audioAnalysis.tox v0.1.0 with Custom Parameters"
   git push
   ```

## Pitfalls

- **Expression typed as a number:** if the field is not blue, it is a constant. Click the mode button until it turns blue, then retype.
- **Parameter name rejected:** names must be a capital letter followed by lowercase letters or digits. `LowCrossover` fails; `Lowcrossover` works.
- **Outputs in the wrong order:** Out CHOP order follows their vertical position inside the component. Move them, then check the connectors again.
- **Works in the host, errors in a blank project:** something inside still points outside. See step 10's fix.
- **Edited the component but the .tox is old:** the .tox only updates when you save it again (step 9).

## What this sets up

- S4 adds OSC output, sending `/audio/low`, `/audio/mid`, `/audio/high`, `/audio/env`, and `/audio/beat` from these outputs.
- S5's operator panel drives the Custom Parameters you just made.

## Docs and tutorials

- [Component](https://docs.derivative.ca/Component), [Base COMP](https://docs.derivative.ca/Base_COMP), [In CHOP](https://docs.derivative.ca/In_CHOP), [Out CHOP](https://docs.derivative.ca/Out_CHOP).
- [Custom Parameters](https://docs.derivative.ca/Custom_Parameters), including the naming rules and the Component Editor.
- [Text DAT](https://docs.derivative.ca/Text_DAT), [Math CHOP](https://docs.derivative.ca/Math_CHOP) (Mult-Add page), [Trigger CHOP](https://docs.derivative.ca/Trigger_CHOP).
- [Derivative TouchDesigner Curriculum, 100 Series](https://learn.derivative.ca/courses/100-fundamentals/), lesson 106 covers COMPs and interface building.
