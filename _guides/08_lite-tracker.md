---
layout: page
title: How the Lite Tracker Works
order: 8
---

**Lite** is YORU Tracker's default tracker. Knowing what it does in each frame makes its settings, and its mistakes, easier to understand.

Lite is an **online** tracker: it uses only the current frame and what it remembers of earlier ones, runs no neural network of its own, and never revises a decision. That is why it can sit in a closed loop, and why it takes well under a millisecond per frame.

---

## One frame, step by step

1. **Predict.** Every track's centre is predicted with a Kalman filter on position and velocity. Time is counted in frames, so frames a live camera dropped are simply a longer step.

2. **Sort the detections.** A box of another class on an animal already reported more confidently is set aside (`duplicate_iou`). The rest are **trusted** (confidence at least `high_confidence`) or **doubtful**.

3. **Primary association.** Every confirmed track, visible or lost, is matched against the trusted detections. A pair is allowed only within `max_distance` of the prediction; its cost comes from the distance, the overlap of the (oriented) boxes and the agreement of their long axes. The cheapest overall assignment is found with the Hungarian method.

4. **Recovery association.** Tracks still unmatched are tried against what is left, with a gate that widens the longer they have been missing (up to `recovery_gate_scale × max_distance`) — but not for a track lost inside a box another track holds: that animal is hidden there (`hidden_guard`).

5. **Doubtful association.** A track seen a frame ago and still unmatched may take a doubtful detection that overlaps its predicted box (`low_confidence_iou`).

6. **Newcomers.** Trusted detections that no track took start **tentative** tracks. A tentative track gets an ID after `min_hits` sightings.

7. **Lost and retired.** Unmatched tracks become **lost**: still predicted, still recoverable. Past `max_age` frames they are **retired** — unless the number of animals is known (below).

```text
             min_hits sightings                 not detected
 tentative  ──────────────────▶  active  ─────────────────────▶  lost
 (no ID)        "created"          ▲              "lost"           │
                                   │                               │ max_age frames
                                   └───────── "recovered" ─────────┤ (population unknown)
                                                                   ▼
                                                                retired
                                                               "retired"
```

---

## Known number of animals

With `population` set (for example, `2` for a pair of flies in a closed chamber):

- **No ID is ever retired**, and no more IDs than `population` are given out.
- When every ID is taken, a candidate seen `min_hits` times is handed to the **nearest lost track**, together with the motion it has followed since it appeared. An animal that jumped, or that reappears after another covered it, gets its own ID back.
- A hidden track waits instead, and a candidate overlapping a tracked animal is not handed over: that is far more often a second box on that animal than an animal that jumped.

On a real recording of two flies (9002 frames, including several minutes of copulation during which the detector reports one box for the pair), the default Lite settings gave 38 IDs; with `population: 2`, exactly 2.

---

## Crowded arenas: spare boxes

Where animals crowd, detectors add boxes that are not new animals: a weaker box around two touching animals already reported one by one, or a second box of another class on the same animal. Three settings keep these from starting tracks or taking IDs:

| Setting | What it stops |
|---|---|
| `high_confidence` | A doubtful box never starts a track, brings a lost one back, or takes a lost ID. It may only continue a track seen a frame ago, right where it is. |
| `duplicate_iou` | Two boxes of different classes on one animal count once (only while `class_aware` is off). |
| `hidden_guard` | The ID of an animal hidden in another's box waits for it, and a box overlapping a tracked animal is never handed to a lost ID. |

On a recording of sixty flies in a closed arena (9000 frames, a YOLOv5 model that adds a weaker box around touching flies and keeps `fly` and `wing_extension` boxes on the same fly), the version-1 settings gave 795 IDs and the current defaults 419. With `population: 60` there are 60 either way, but jumps of over 25 px in one frame — the mark of a track taking another fly's detection — fell from 90 to 4.

---

## Details worth knowing

- **Jumps.** A lost track found again where its motion model gave it less than a 1 % chance (the animal jumped, or stopped while unseen) starts its motion afresh there, rather than taking the distance for speed. The velocity it reports stays the animal's.
- **Orientation.** Rectangles have no head and tail, so orientation is compared as an axis (modulo 180°), and only for elongated boxes.
- **Oriented boxes** from OBB models are used throughout: overlap is computed between the rotated boxes.
- **Determinism.** The same detections in the same order always give the same IDs, whether they come from a camera, a video or a file. This is tested.

---

## What Lite cannot do (yet)

Most remaining ID switches are **animals in contact that the detector reports as one box**: which animal leaves the box on which side cannot be told from position alone. Setting `population` helps a lot, but cannot always tell two identical-looking animals apart.

That is the job of the planned **Advanced** tracker, which will re-identify animals by appearance when identities are ambiguous. Explicit occlusion handling for Lite and offline refinement (gap filling, smoothing) are also planned.
