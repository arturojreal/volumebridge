# S2 runbook - Audio analysis deep dive

Tuesday Nov 10 (started early on Mon Nov 9 is fine). About 3 hours. Machine: [M/B], either works. Build inside `td/host/volumeBridgeHost.toe`, continuing from S1. Node names match `td/NAMING.md`. Parameter names marked **[verify]** are from memory; if one is labeled slightly differently in your build, use the closest match.

## Goal

Turn the S1 audio chain into clean control signals: three band energies (low, mid, high) and one envelope, each smoothed and scaled 0 to 1. These are exactly the values S4 will send as `/audio/low`, `/audio/mid`, `/audio/high`, and `/audio/env`.

**Done when:**
- Three band channels (`low`, `mid`, `high`) and one envelope channel (`env`) update live.
- Music moves all three bands; voice mostly moves `mid`; a kick drum or bass mostly moves `low`.
- Values sit between 0 and 1 and do not jitter.
- The project is saved and committed.

## Before you start (5 min)

1. `git pull` in your clone, then open `td/host/volumeBridgeHost.toe`.
2. Your S1 setup: default audio device at 44100 Hz. That rate means frequencies up to about 22,000 Hz are available, which covers every band below.
3. Have two test sources ready: a music track with clear bass, and your own voice (or a podcast).

## The plan in one picture

```
audioIn -> mono -> lowFilter            -> lowLevel  -> lowSmooth  -> lowNorm  -> low
                -> midHigh -> midFilter -> midLevel  -> midSmooth  -> midNorm  -> mid
                -> highFilter           -> highLevel -> highSmooth -> highNorm -> high
           level (from S1) -> envSmooth -> envNorm -> env
low + mid + high -> bandMerge -> out_bands        env -> out_env
```

Each band is: filter the audio to that frequency range, measure its loudness (RMS), smooth it, scale it to 0 to 1.

## Step 1 - Make the signal mono (10 min)

Stereo doubles every channel downstream and makes bands confusing to read. Average left and right first.

1. Add a **Math CHOP** after `audioIn`. Rename it `mono`.
2. On its **OP** page, set **Combine Channels** to **Average**. The output is now one channel.

## Step 2 - Split into three bands (35 min)

Crossovers: low is below 250 Hz, mid is 250 to 4000 Hz, high is above 4000 Hz. Use **Audio Filter CHOPs**, which filter the raw audio before you measure it.

1. **Low:** add an **Audio Filter CHOP** after `mono`, rename it `lowFilter`. Set **Filter** to **Low Pass** and the cutoff to **250** Hz. **[verify]** the cutoff parameter's name and that its unit is Hz.
2. **High:** add another **Audio Filter CHOP** after `mono`, rename it `highFilter`. Set **Filter** to **High Pass**, cutoff **4000** Hz.
3. **Mid:** chain two filters so the band has clean edges:
   - Add an **Audio Filter CHOP** after `mono`, rename it `midHigh`: **High Pass** at **250** Hz.
   - Add another after `midHigh`, rename it `midFilter`: **Low Pass** at **4000** Hz.
4. Play music. All three filter viewers should show audio, with `lowFilter` looking slow and smooth and `highFilter` looking fast and fizzy.

Why the chain for mid: a High Pass then a Low Pass gives you exact control of both crossovers, and they line up with the low and high cutoffs so the bands neither overlap nor leave a gap.

## Step 3 - Measure each band's loudness (20 min)

1. Add an **Analyze CHOP** after each band filter: `lowLevel`, `midLevel`, `highLevel`.
2. Set each one's **Function** to **RMS Power**, the same as `level` in S1.
3. Play music. Each band now outputs a single number that rises and falls. Notice the scales differ: `lowLevel` is usually much bigger than `highLevel`. Step 5 fixes that.

## Step 4 - Smooth the jitter (25 min)

Raw RMS flickers frame to frame. A **Lag CHOP** smooths it, with separate times for rising and falling.

1. Add a **Lag CHOP** after each level: `lowSmooth`, `midSmooth`, `highSmooth`.
2. The **Lag** parameter has two fields: the first is the rise time, the second the fall time, in seconds.
3. Starting values:
   - `lowSmooth`: `0.02` and `0.25` (punchy attack, bass that hangs a moment).
   - `midSmooth`: `0.02` and `0.15`.
   - `highSmooth`: `0.01` and `0.1` (highs are fast and should stay fast).
4. Compare the Lag output against its input with music playing. An Analyze CHOP outputs one value per frame, so its viewer and the Lag's viewer only show a single number moving up and down, with no history. To see the shape over time:
    - Add a **Trail CHOP** after `lowLevel` and another after `lowSmooth`, named `lowLevelTrail` and `lowSmoothTrail`. Each one records the last few seconds (set **Window Length** if you want more or less).
    - Put the two side by side and enlarge their viewers. The raw trail looks spiky; the smooth trail should follow the same peaks with the flicker taken out.
    - Repeat for mid and high, or move the two Trails onto the other bands as you tune them. They are only for viewing, so delete them when you are done or keep them off to the side.
    - Too slow feels laggy on the wall; too fast flickers.

## Step 5 - Scale each band to 0 to 1 (25 min)

Each band's smoothed level tops out at a different value (low might peak near 0.3, high near 0.02). A Math CHOP stretches each band so its loudest moments reach 1. To do that, you first measure each band's peak, the highest value it reaches during loud music, using two temporary measuring nodes.

**5a. Build the peak meter (one time, 5 min)**

1. Add a **Trail CHOP**. Rename it `peakTrail`. On its parameters, set **Window Length** to `20` seconds, so it remembers the last 20 seconds.
2. Add an **Analyze CHOP**. Rename it `peakMax`. Wire `peakTrail`'s output into `peakMax`'s input. Set `peakMax`'s **Function** to **Maximum**.
3. `peakMax` now outputs one number: the highest value that reached `peakTrail` in the last 20 seconds. Read it from `peakMax`'s node viewer.

**5b. Measure the low band (5 min)**

1. Wire `lowSmooth`'s output into `peakTrail`'s input.
2. Play the loudest part of your test track, at the volume you will normally use, for at least 20 seconds.
3. Read `peakMax`. Write that number down as the **low peak** (for example `0.31`).
4. Multiply it by `0.9` and write that down too (for example `0.31 x 0.9 = 0.28`). This is the **low scale value**. Using 90 percent of the peak means loud moments actually reach 1 instead of stopping just under it.

**5c. Measure mid and high (5 min)**

1. Drag the wire into `peakTrail` so it comes from `midSmooth` instead. Repeat 5b steps 2 to 4 and write down the **mid scale value**.
2. Do the same with `highSmooth` and write down the **high scale value**.
3. Keep `peakTrail` and `peakMax` for step 7; you will measure the envelope with them too.

**5d. Add the scaling nodes (10 min)**

1. Add a **Math CHOP**. Rename it `lowNorm`. Wire `lowSmooth`'s output into `lowNorm`'s input.
2. On `lowNorm`, open the **Range** page:
    - **From Range**: first field `0`, second field your **low scale value** (for example `0.28`).
    - **To Range**: first field `0`, second field `1` (these are the defaults; leave them).
3. Repeat for mid: a Math CHOP named `midNorm`, fed from `midSmooth`, **From Range** `0` and your **mid scale value**, **To Range** `0` and `1`.
4. Repeat for high: a Math CHOP named `highNorm`, fed from `highSmooth`, **From Range** `0` and your **high scale value**, **To Range** `0` and `1`.
5. Check: with the loud part of the track playing, each `...Norm` node should reach roughly 1 on its loudest moments and sit near 0 in silence.

**5e. Optional: stop values going above 1 (5 min)**

Very loud moments can still push a band above 1. If you want a hard ceiling:

1. Add a **Limit CHOP** after `lowNorm`, renamed `lowClamp`. Set **Type** to **Clamp**, **Minimum** to `0`, **Maximum** to `1`.
2. Do the same after `midNorm` (`midClamp`) and `highNorm` (`highClamp`).
3. If you add these, the next step continues from the `...Clamp` nodes. If you skip them, it continues from the `...Norm` nodes.

S3 replaces these hand-measured numbers with a Gain parameter you can adjust live.

## Step 6 - Name the channels (10 min)

Every node so far outputs a channel with a generic name (usually `chan1`). The channel names become OSC addresses in S4, so give each band its real name now.

1. Add a **Rename CHOP**. Rename the node `lowName`. Wire the last low node into it (`lowClamp` if you made it, otherwise `lowNorm`).
2. On `lowName`, set **Rename From** to `*` (meaning "whatever the channel is called now") and **Rename To** to `low`.
3. Add a Rename CHOP named `midName`, fed from the last mid node, **Rename From** `*`, **Rename To** `mid`.
4. Add a Rename CHOP named `highName`, fed from the last high node, **Rename From** `*`, **Rename To** `high`.
5. Check: click each `...Name` node. Its viewer should label the channel `low`, `mid`, or `high`.

## Step 7 - The envelope (25 min)

The envelope is the overall loudness of the whole mix, smoothed with a fast rise and a slow fall, so it swells with the music instead of jumping on every hit. You build it from `level`, the full-mix RMS node you made in S1 (Analyze CHOP, Function set to RMS Power).

1. **Smooth it.** Add a **Lag CHOP**. Rename it `envSmooth`. Wire `level`'s output into `envSmooth`'s input. Set its **Lag** fields to `0.01` (first field, rise) and `0.4` (second field, fall).
2. **Measure its peak** with the meter from step 5:
    - Drag the wire into `peakTrail` so it comes from `envSmooth`.
    - Play the loudest part of your test track for at least 20 seconds.
    - Read `peakMax` and write it down as the **envelope peak**.
    - Multiply by `0.9` and write that down as the **envelope scale value**.
3. **Scale it.** Add a **Math CHOP**. Rename it `envNorm`. Wire `envSmooth`'s output into `envNorm`'s input. On its **Range** page set **From Range** to `0` and your **envelope scale value**; leave **To Range** at `0` and `1`.
4. **Optional ceiling.** Add a **Limit CHOP** after `envNorm`, renamed `envClamp`, with **Type** **Clamp**, **Minimum** `0`, **Maximum** `1`.
5. **Name it.** Add a **Rename CHOP**. Rename it `envName`. Wire the last envelope node into it (`envClamp` or `envNorm`). Set **Rename From** to `*` and **Rename To** to `env`.
6. **Tidy up.** You are done measuring. Delete `peakTrail` and `peakMax`, or drag them off to the side for later re-tuning.
7. Check: with music playing, `envName` rises quickly when the music gets loud and falls slowly, over about half a second, when it gets quiet.

## Step 8 - Clean outputs (10 min)

1. Add a **Merge CHOP**. Rename it `bandMerge`. Wire `lowName` into its first input, `midName` into its second, and `highName` into its third. The input order sets the channel order, so keep it low, mid, high.
2. Check: `bandMerge`'s viewer shows three channels named `low`, `mid`, `high`.
3. Add a **Null CHOP**. Rename it `out_bands`. Wire `bandMerge` into it.
4. Add another **Null CHOP**. Rename it `out_env`. Wire `envName` into it.

A Null passes its input through unchanged. S3 and S4 read from `out_bands` and `out_env`, so you can rework anything upstream later without breaking what reads from them.

## Step 9 - Test against the done list (15 min)

Watch `out_bands` and `out_env` while you play each source:

1. **Music with clear bass:** all three bands move. `low` jumps toward 1 on kicks or bass notes.
2. **Your voice (talk normally near the mic, or play a podcast):** `mid` moves the most. `low` and `high` stay noticeably lower.
3. **Silence:** all four channels settle close to 0 and stay steady, without flicker.
4. **If a test fails, adjust one thing at a time:**
    - A band never reaches about 1 on loud music: lower that band's scale value in its `...Norm` node (From Range second field).
    - A band sits at 1 most of the time: raise that scale value.
    - A channel flickers: raise the second (fall) value on its `...Smooth` Lag CHOP, for example from `0.15` to `0.25`.
    - A channel feels late: lower the first (rise) value on its Lag CHOP, but keep it at `0.01` or above.

## Step 10 - Stretch: beat detection (optional, 20 min)

This fires a short pulse on each kick drum, using the low band.

1. Add a **Trigger CHOP**. Rename it `beatTrigger`. Wire `lowNorm` into it (the scaled low band, before its Rename).
2. Set its **Trigger Threshold** to `0.6` **[verify]**: the pulse fires when `low` crosses 0.6.
3. Keep the pulse short: set its attack and release lengths to `0`, and its decay length to about `0.1` seconds **[verify parameter names]**.
4. Add a **Rename CHOP** named `beatName`, fed from `beatTrigger`, **Rename From** `*`, **Rename To** `beat`.
5. Add a **Null CHOP** named `out_beat`, fed from `beatName`.
6. Check with music: one pulse per kick. If it fires twice per kick, raise the threshold (for example to `0.7`). If it misses kicks, lower it (for example to `0.5`).

## Step 11 - Save, log, commit (10 min)

1. Save the project with Ctrl+S.
2. In your tracking log, record:
    - the four scale values you used (low, mid, high, envelope),
    - your final Lag values per band,
    - whether you did the clamps and the beat stretch,
    - anything that confused you.
3. Commit and push:
   ```
   git add -A
   git commit -m "S2: band analysis, envelope, and clean outputs in volumeBridgeHost.toe"
   git push
   ```

## Pitfalls

- **Everything doubled:** you skipped the mono step, so each band has two channels.
- **One band dead:** a filter set to the wrong type (Low Pass where you meant High Pass), or the cutoff unit is not Hz.
- **Bands look the same:** the filters are wired from the wrong node; each must come from `mono` (or `midHigh` for `midFilter`).
- **Jittery output:** Lag times too short. Increase the fall time first.
- **Laggy on the wall later:** Lag rise times too long. Keep rises at or under 0.03 seconds.
- **Values over 1:** a From Range peak set too low for the current volume. Re-measure, or add the Limit CHOP clamp.

## What this sets up

- S3 wraps this network into `td/audioAnalysis.tox`, with Custom Parameters for gain, smoothing, and crossovers in place of the hand-set numbers.
- S4 sends `out_bands` and `out_env` as `/audio/low`, `/audio/mid`, `/audio/high`, and `/audio/env`.

## Docs and tutorials

- Operators used today: [Math CHOP](https://docs.derivative.ca/Math_CHOP) (Combine Channels, Range page), [Audio Filter CHOP](https://docs.derivative.ca/Audio_Filter_CHOP), [Analyze CHOP](https://docs.derivative.ca/Analyze_CHOP), [Lag CHOP](https://docs.derivative.ca/Lag_CHOP), [Trail CHOP](https://docs.derivative.ca/Trail_CHOP), [Limit CHOP](https://docs.derivative.ca/Limit_CHOP), [Rename CHOP](https://docs.derivative.ca/Rename_CHOP), [Merge CHOP](https://docs.derivative.ca/Merge_CHOP), [Null CHOP](https://docs.derivative.ca/Null_CHOP), [Trigger CHOP](https://docs.derivative.ca/Trigger_CHOP).
- [Creating Audio with CHOPs (Derivative)](https://derivative.ca/UserGuide/Creating_Audio_with_CHOPs), an overview of the audio operator family.
- [IIHQ: Audio reactive visuals, a beginner guide](https://interactiveimmersive.io/blog/touchdesigner-lessons/audio-reactive-visuals-a-beginner-guide/), for more band-splitting patterns.
