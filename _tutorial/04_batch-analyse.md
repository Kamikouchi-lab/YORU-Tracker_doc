---
layout: page
order: 4
title: Step4. Track All Videos and Analyse
---

## Track every video

### In the window

1. Press **Batch Tracking** (or **Go → Batch Tracking**).
2. **Video folder** → `D:\experiment\videos`.
3. **Detector model** → the same model as before. **Conf.** as before.
4. **Settings...** → **Load...** → `two_flies.yaml` → **Apply**.
5. **Output folder** → `D:\experiment\results`.
6. Press **Run batch**, and watch the *Files* table. A file that fails is marked `failed` with its error, and the batch goes on; the **Failure report** lists every failure at the end.

### Or on the command line

```
uv run yoru-tracker batch D:\experiment\videos --model D:\models\best.pt --config D:\experiment\two_flies.yaml --out D:\experiment\results
```

Either way, each video gets three files:

```text
D:\experiment\results\
├── pair01_tracks.csv        one row per fly per frame
├── pair01_tracks.json       settings, model, versions: how to reproduce it
├── pair01_detections.csv    the detector's output, to re-track without the model
├── pair02_tracks.csv
└── ...
```

---

## Analyse the tracks

The tracks CSV is a plain table; its columns are described in [Output Files]({{ site.baseurl }}/guides/06-output-files/). pandas and matplotlib are already in the YORU Tracker environment, so a script can be run with `uv run python analyse.py`:

```python
# analyse.py
import json

import matplotlib.pyplot as plt
import pandas as pd

name = r"D:\experiment\results\pair01"
tracks = pd.read_csv(f"{name}_tracks.csv")
fps = json.load(open(f"{name}_tracks.json", encoding="utf-8"))["source"]["fps"]

# Position of each fly, one column per track ID
cx = tracks.pivot(index="frame_id", columns="track_id", values="cx")
cy = tracks.pivot(index="frame_id", columns="track_id", values="cy")

# Distance between the two flies, in pixels, per frame
distance = ((cx[0] - cx[1]) ** 2 + (cy[0] - cy[1]) ** 2) ** 0.5

# Speed of each fly, in pixels per second
tracks["speed"] = (tracks.vx ** 2 + tracks.vy ** 2) ** 0.5 * fps
print(tracks.groupby("track_id").speed.describe())

# Time each fly spent in each class (e.g. solo / copulation), in seconds
print(tracks.groupby(["track_id", "class_name"]).size().unstack(fill_value=0) / fps)

# Trajectories
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(11, 4))
for track_id, t in tracks.groupby("track_id"):
    ax1.plot(t.cx, t.cy, lw=0.7, label=f"ID {track_id}")
ax1.invert_yaxis()                  # image coordinates: y grows downwards
ax1.set_aspect("equal")
ax1.legend()
ax2.plot(distance.index / fps, distance)
ax2.set_xlabel("time (s)")
ax2.set_ylabel("distance between flies (px)")
plt.tight_layout()
plt.show()
```

<div class="note" markdown="1">
**Frames without a row.** By default, a fly that was not detected in a frame has no row in that frame, so its position is `NaN` there after `pivot`. To have a predicted position instead, export with *Include predicted rows of lost tracks* (`--include-predicted`); those rows have `predicted` = 1.
</div>

---

## Changing your mind later

Because the detections were saved, the whole experiment can be tracked again with different settings — in seconds per video, without the model or a GPU:

```
uv run yoru-tracker track D:\experiment\results\pair01_detections.csv --config new_settings.yaml --out D:\experiment\results_v2
```

The `*_tracks.json` beside each result records the exact settings it was made with.
