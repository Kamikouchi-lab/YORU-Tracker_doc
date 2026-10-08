---
layout: page
title: Tracker Settings
order: 5
---

The tracker settings decide how detections become tracks. They are shared by Video, Realtime and Batch Tracking, saved as a versioned YAML file, and written into the JSON beside every result, so a run can always be repeated.

Open the settings window with **Settings** on the start screen, **Settings...** on any workflow screen, or **Tracker → Settings...**.

---

## The settings window

| Part | |
|---|---|
| **Mode** | The tracker: `lite` (the default) or `baseline`. The line below describes the chosen one. `advanced` is listed as *Planned*. |
| **Settings** | The settings worth changing day to day (below). Hover over a setting for its explanation. |
| **More** | The rest, folded away. |
| **Apply** | Use the settings shown. Invalid values are refused with the reason, and nothing changes. |
| **Load...** / **Save...** | Read / write a YAML (or JSON) settings file. A loaded file is shown, not used, until you press **Apply**. |
| **Defaults** | Show the default settings. Press **Apply** to use them. |
| **Close** | Close the window. Unapplied changes are discarded. |

Applying does not re-run anything by itself. What it means depends on the screen:

| Screen | New settings reach it when you |
|---|---|
| Video Tracking | press **Re-track** (or **Run tracking** again). |
| Realtime Tracking | press **Reset tracker** (IDs restart at 0) or start again. |
| Batch Tracking | press **Run batch**. |

---

## The main settings

| In the window | Key | Default | Meaning |
|---|---|---|---|
| Number of animals (0 = ?) | `lifecycle.population` | `0` | How many animals are **always** in view, if known — e.g. `2` for a pair of flies in a chamber. Then no ID is ever retired or added beyond this number, and an animal that jumps, or reappears after another covered it, gets its own ID back. `0`: animals may enter and leave. |
| Max age (frames) | `lifecycle.max_age` | `10` | Frames a track may go undetected and keep its ID. After that it is retired. |
| Min hits | `lifecycle.min_hits` | `2` | Detections a newcomer needs before it gets an ID, so a one-frame false detection never uses up a number. `1` gives an ID at first sight. |
| Max distance (px) | `association.max_distance` | `100` | How far, in pixels, a detection may be from where a track is predicted to be. **Scale it to your magnification.** |
| Trusted confidence (>=) | `association.high_confidence` | `0.5` | Detections less confident than this never start a track, bring one back or take a lost ID; they only continue a track seen a frame ago, right where it is. This keeps out the weaker box a detector adds around two touching animals it has already reported one by one. `0`: trust every detection. |
| Distance weight | `association.distance_weight` | `1.0` | Cost of the distance between a detection and a track's predicted centre. |
| IoU weight | `association.iou_weight` | `1.0` | Cost of poor overlap between the boxes. |
| Axis weight | `association.axis_weight` | `0.25` | Cost of disagreeing long axes. Only for elongated boxes; head and tail are not told apart. |
| Match within class only | `association.class_aware` | off | Never give a track a detection of another class. Keep it **off when classes are behaviours** (`solo`, `copulation`) — an animal changes behaviour, not identity. Turn it **on when classes are different animals**. |
| Recovery pass for lost tracks | `association.recovery` | on | Look for lost tracks again with a gate that widens the longer they have been missing. |
| Keep hidden animals' IDs in place | `association.hidden_guard` | on | An animal lost inside another's box (two seen as one) is hidden there: its ID waits for it instead of jumping to a detection elsewhere, and a box overlapping a tracked animal is never handed to a lost ID. |
| Motion prediction (Kalman) | `kalman.enabled` | on | Predict where each track moves. Off: expect it where it was last seen. |

## More

| In the window | Key | Default | Meaning |
|---|---|---|---|
| Size weight | `association.size_weight` | `0.0` | Cost of differing box areas. |
| Min IoU | `association.min_iou` | `0.0` | Refuse a pair with less overlap than this whose centres are more than half a body length apart. `0`: off. |
| Min IoU, doubtful detection | `association.low_confidence_iou` | `0.5` | How much a detection below the trusted confidence must overlap a track's predicted box to continue it. |
| Same animal, other class IoU | `association.duplicate_iou` | `0.5` | Two boxes of different classes overlapping more than this are one animal, and the more confident is used. Only while *Match within class only* is off. `0`: off. |
| Recovery gate (x max dist.) | `association.recovery_gate_scale` | `3.0` | How far the recovery gate may widen, in multiples of *Max distance*. |
| Process noise (x size) | `kalman.process_noise` | `0.2` | Expected acceleration, relative to the box size. |
| Measurement noise (x size) | `kalman.measurement_noise` | `0.1` | Detector jitter, relative to the box size. |
| Log every track event | `log_events` | off | Write every `created` / `lost` / `recovered` / `retired` event to the log file. |

The cost of matching a detection to a track is `distance_weight × distance/gate + iou_weight × (1 − IoU) + axis_weight × axis + size_weight × size`, each term between 0 and 1. A pair costing `distance_weight + iou_weight` or more is never made. None of these weights is a universal constant: compare settings with the [Benchmark]({{ site.baseurl }}/guides/09-benchmark/) before changing them.

The Kalman noise levels are relative to the box size, so one setting serves a 20-pixel fly and a 200-pixel mouse. Their ratio matters most: with a process noise far above the measurement noise, the velocity follows every jitter of the detector, and animals crossing at speed swap IDs.

---

## Which settings to change

Start from the defaults, track a short video in [Video Tracking]({{ site.baseurl }}/guides/02-video-tracking/), and change one thing at a time:

1. **Is the number of animals fixed?** Set **Number of animals**. This is the most effective setting when it applies: the number of IDs can then never exceed it. In a crowded arena especially, give it whenever it is known — without it, a box around two touching animals that the detector scores above *Trusted confidence* can still start a track of its own.
2. **Are the classes behaviours or animals?** Behaviours of one animal: leave *Match within class only* off. Different animals (for example, males and females labelled as separate classes): turn it on.
3. **Do IDs break when an animal moves fast?** Raise **Max distance** — it is in pixels, so it depends on your camera's magnification and frame rate.
4. **Do IDs jump to a neighbouring animal?** Lower **Max distance**.
5. **Is the detector missing animals for a while?** Raise **Max age**, so a track keeps its ID through longer gaps.
6. **Do short-lived IDs appear on false detections?** Raise **Min hits**, or **Trusted confidence**.

After each change, **Apply** and **Re-track**, and look at the number of track IDs and at the frames where IDs are created and lost.

<div class="note" markdown="1">
**Do not tune on one video.** Settings that fix one recording can break others. Check them on a few videos — or on the synthetic scenarios with `yoru-tracker bench --config my_settings.yaml` — before a large batch.
</div>

---

## The settings file

`yoru-tracker config` prints the defaults; **Save...** in the window writes the settings shown.

```
uv run yoru-tracker config > tracker.yaml
```

```yaml
config_version: 2
tracker:
  mode: lite
  lifecycle:
    max_age: 10
    min_hits: 2
    population: 0
  association:
    distance_weight: 1.0
    iou_weight: 1.0
    axis_weight: 0.25
    size_weight: 0.0
    max_distance: 100.0
    min_iou: 0.0
    class_aware: false
    recovery: true
    recovery_gate_scale: 3.0
    high_confidence: 0.5
    low_confidence_iou: 0.5
    duplicate_iou: 0.5
    hidden_guard: true
  kalman:
    enabled: true
    process_noise: 0.2
    measurement_noise: 0.1
  advanced:
    reid: false
    occlusion_recovery: false
  log_events: false
```

A file may give only some keys; the rest take their defaults. For a pair of flies in a chamber:

```yaml
config_version: 2
tracker:
  mode: lite
  lifecycle:
    max_age: 10
    min_hits: 2
    population: 2
  association:
    max_distance: 100.0
```

Use it with **Load...** in the window, or `--config` on the [command line]({{ site.baseurl }}/guides/07-command-line/). JSON files with the same structure are read too.

### Loading is strict

A setting that is wrong is an **error naming it**, never a silent default — a misspelt `max_age` that quietly fell back to 10 would change every ID in the output. All the problems of a file are listed at once:

- an unknown key (`unknown key lifecycle.max_ages`);
- a value of the wrong type, or `.nan` / `.inf`;
- a value out of range:

    | Key | Allowed |
    |---|---|
    | `max_age`, `population` | ≥ 0 |
    | `min_hits` | ≥ 1 |
    | the four weights | ≥ 0, with `distance_weight + iou_weight` > 0 |
    | `max_distance` | > 0 |
    | `min_iou`, `high_confidence`, `low_confidence_iou`, `duplicate_iou` | 0 to 1 |
    | `recovery_gate_scale` | ≥ 1 |
    | `process_noise`, `measurement_noise` | > 0 |

- an unsupported `config_version`;
- an `advanced.*` option with a mode other than `advanced`.

### Version 1 files

`config_version: 1` files were written before `high_confidence`, `duplicate_iou` and `hidden_guard` existed. They are read with these three off (`0`, `0`, `false`), so they track exactly as they did when they were written, and are saved again as version 2.

---

## Tracker modes

| Mode | |
|---|---|
| `lite` | The default. An online tracker: motion prediction, gated matching by distance, overlap and axis, recovery of lost tracks, known-population mode. Runs live. See [How the Lite Tracker Works]({{ site.baseurl }}/guides/08-lite-tracker/). |
| `baseline` | YORU's own frame-to-frame matching, called as it is. For comparison. |
| `advanced` | Planned (appearance-based re-identification). Choosing it is an error saying so — it never falls back to Lite. |

Trackers from installed plugins are listed as well; see [Tracker API]({{ site.baseurl }}/devnotes/01-tracker-api/#modes-and-plugins).
