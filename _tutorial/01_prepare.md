---
layout: page
order: 1
title: Step1. Prepare
---

This protocol takes you from a YORU model and a few videos to an analysed table of per-animal tracks. The example is a **pair of flies in a courtship chamber**, detected with the classes `solo` and `copulation` — but the steps are the same for any animal.

---

## What you need

### 1. YORU Tracker, installed

Follow the [Install guide]({{ site.baseurl }}/guides/01-install/), and check that it starts:

```
cd YORU-dev\YORU-Tracker
uv run yoru-tracker --version
```

### 2. A detector model trained in YORU

YORU Tracker has no detector of its own; it uses the model you trained in YORU (`.pt`, `.pth` or `.onnx`). If you do not have one yet, follow YORU's [Step-by-step Protocols]({{ site.yoru.docs }}tutorial/01-preparation-tutorial/), which train a model on the [Fruit Fly Copulation Dataset](https://zenodo.org/records/15803067).

> The model should detect **each animal**: every animal in every frame, as its own box. How the boxes are classed (by behaviour, by sex, ...) matters less, but decides one setting in Step 3.

### 3. Videos

Put the videos of one experiment into one folder, for example:

```text
D:\experiment\videos\
├── pair01.mp4
├── pair02.mp4
└── pair03.mp4
```

---

## Know three numbers before you start

| Number | Why | Setting |
|---|---|---|
| **How many animals** are in view, if it never changes (here: 2) | The single most effective setting: IDs can then never exceed it. | Number of animals (`population`) |
| **How far an animal moves** in one frame at most, in pixels | Detections further than this from a track's prediction cannot be matched to it. | Max distance (`max_distance`) |
| **Whether classes are behaviours or animals** | Behaviours (`solo`, `copulation`): an animal changes class but not identity. Animals (`male`, `female`): a class never changes. | Match within class only (`class_aware`) |

To measure distances in pixels, open a frame in any image viewer and read the coordinates of a fly at two moments, or look at the `vx`/`vy` columns of a first tracking run (Step 2).

YORU's own video analysis files are named after the video (`pair01.mp4` → `pair01.csv`). If you already have them, or real-time recordings, see [Step 5]({{ site.baseurl }}/tutorial/05-yoru-recordings/): they can be tracked without running the detector again.
