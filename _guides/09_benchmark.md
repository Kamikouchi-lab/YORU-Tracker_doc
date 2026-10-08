---
layout: page
title: Benchmark
order: 9
---

`yoru-tracker bench` scores trackers on **synthetic behavioural scenarios** — fly-sized animals, with detector jitter and misses, and a known ground truth. Use it to see what a tracker does well and badly, and to check your own settings before relying on them.

```
uv run yoru-tracker bench                              # Lite and Baseline, every scenario
uv run yoru-tracker bench --config my_settings.yaml    # also your settings, as "custom"
uv run yoru-tracker bench --scenario dense_arena --json bench.json
```

Every Lite configuration that does not set `population` is also scored as **`<name>+N`**: told the number of animals, on the scenarios where it is fixed.

---

## Scenarios

| Scenario | What it tests |
|---|---|
| `single` | One animal, random walk. |
| `far_apart` | Four animals, each in its own quadrant. |
| `fast_crossing` | Two animals cross at about 9 px/frame. |
| `parallel_motion` | Two animals walk side by side, 32 px apart. |
| `close_approach` | Two animals meet head to head, pause, and separate. |
| `courtship` | One animal follows and circles another at about 1.3 body lengths. |
| `contact` | Two animals touch for about 20 frames; the detector sees one box. |
| `temporary_overlap` | Two animals pass over each other, both still detected. |
| `complete_overlap` | One animal lies on another for about 15 frames, then both leave. |
| `detector_dropout` | Three animals, each missed for bursts of 3–6 frames. |
| `animal_entry` | Two animals present; two more walk in at frames 60 and 120. |
| `animal_exit` | Four animals; two walk out of the arena. |
| `high_density` | Twelve animals in one arena, with frequent near encounters. |
| `obb_crossing` | Two animals cross diagonally; oriented-box detector. |
| `jump` | An animal jumps 270 px (about 7 body lengths) in one frame. |
| `long_contact` | Two animals stay in contact for about 70 frames (longer than `max_age`), seen as one box. |
| `spanning_box` | Two touching pairs, each animal detected, and often a low-confidence box around the pair. |
| `class_duplicates` | Four animals; each is often reported a second time, as another class. |
| `hidden_in_merge` | An animal hidden in another's box while a confident box spans two others elsewhere. |
| `dense_arena` | Forty animals in one arena: merges, spanning boxes and boxes of a second class. |

---

## Columns

| Column | Meaning | Better |
|---|---|---|
| `IDSW` | ID switches: an animal matched to a different ID than in its last matched frame. | lower |
| `frag` | Fragmentations: an animal's matched run interrupted and resumed, by any ID. | lower |
| `newID` | False new IDs: an animal that already had an ID picked up a brand-new one. | lower |
| `recov` | Recoveries / opportunities: of the resumed runs, how many came back with the same ID as before. | higher |
| `IDF1` | ID F1: the share of outputs and ground truth that agree under the best one-to-one mapping between IDs and animals over the whole run (0–1). | higher |
| `MOTA` | 1 − (misses + false positives + ID switches) / ground-truth objects. | higher |
| `recall`, `prec` | Matched / ground-truth objects, and matched / tracker outputs. | higher |
| `IDs` / `GT` | IDs given / animals in the ground truth. | close |
| `ms/f` | Tracker time per frame, in milliseconds. | lower |

An output and a ground-truth animal count as the same animal in a frame when their centres are close; a pairing is kept from frame to frame while it stays close.

The table ends with each tracker's totals and its mean time per frame.

---

## Results (seed 0)

*Baseline* is YORU's existing frame-to-frame matching (`match_to_previous`), called directly. *Lite+N* is Lite told the number of animals.

| Scenario | Lite IDSW | Lite+N IDSW | Baseline IDSW | Lite IDF1 | Lite+N IDF1 | Baseline IDF1 |
|---|---:|---:|---:|---:|---:|---:|
| single | 0 | 0 | 2 | 0.995 | 0.995 | 0.472 |
| far_apart | 0 | 0 | 6 | 0.996 | 0.996 | 0.708 |
| fast_crossing | 0 | 0 | 4 | 0.997 | 0.997 | 0.484 |
| parallel_motion | 0 | 0 | 7 | 0.991 | 0.991 | 0.537 |
| close_approach | 0 | 0 | 7 | 0.991 | 0.991 | 0.472 |
| courtship | 0 | 0 | 5 | 0.994 | 0.994 | 0.528 |
| contact | 2 | 2 | 3 | 0.517 | 0.519 | 0.519 |
| temporary_overlap | 0 | 0 | 5 | 0.996 | 0.996 | 0.527 |
| complete_overlap | 1 | 2 | 5 | 0.758 | 0.564 | 0.429 |
| detector_dropout | 0 | 0 | 13 | 0.973 | 0.973 | 0.339 |
| animal_entry | 0 | 0 | 0 | 0.998 | 0.998 | 1.000 |
| animal_exit | 0 | 0 | 4 | 0.997 | 0.997 | 0.863 |
| high_density | 0 | 0 | 59 | 0.987 | 0.987 | 0.472 |
| obb_crossing | 0 | 0 | 6 | 0.995 | 0.995 | 0.470 |
| jump | 1 | 0 | 1 | 0.751 | 0.999 | 0.750 |
| long_contact | 1 | 0 | 1 | 0.725 | 0.895 | 0.724 |
| spanning_box | 2 | 0 | 6 | 0.953 | 0.995 | 0.662 |
| class_duplicates | 0 | 0 | 93 | 0.996 | 0.996 | 0.139 |
| hidden_in_merge | 11 | 0 | 11 | 0.644 | 0.960 | 0.433 |
| dense_arena | 63 | 51 | 218 | 0.735 | 0.753 | 0.402 |
| **total** | **81** | **55** | **456** | | | |

Lite takes 0.1–0.5 ms per frame on the small scenarios and about 2 ms with forty animals — small next to any detector.

Most remaining switches are animals in contact that the detector reports as one box: which animal leaves the box on which side cannot be told from position alone. In `hidden_in_merge` without the number of animals, a confident box around two touching animals starts a track of its own; with `population` it cannot.

Before the spare-box handling (`high_confidence`, `duplicate_iou`, `hidden_guard`), the last four scenarios gave Lite 8, 58, 11 and 101 switches, and Lite+N 0, 0, 6 and 96.

<div class="note" markdown="1">
Synthetic scenarios are a check, not a guarantee. Confirm settings on your own recordings too: look at the number of IDs, and at the frames where IDs are created and lost, in [Video Tracking]({{ site.baseurl }}/guides/02-video-tracking/).
</div>
