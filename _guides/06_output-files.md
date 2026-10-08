---
layout: page
title: Output Files
order: 6
---

For a video `movie.mp4`, YORU Tracker writes:

| File | Written by | Content |
|---|---|---|
| `movie_tracks.csv` | Export CSV, Batch, `track`, `batch` | One row per track per frame. |
| `movie_tracks.json` | always with the CSV | Everything needed to reproduce the CSV. |
| `movie_detections.csv` | *Save detections* (on by default), Batch, `track`, `batch` | The detector's output per frame, for re-tracking without the model. |
| `movie_tracked.mp4` | Render video, `--render` | The video with the tracks drawn on it. |

A live recording writes `live_YYYYMMDD_HHMMSS_tracks.csv` and its `.json` (`..._part2_tracks.csv` after a tracker reset). Re-tracking a detections CSV from the command line writes `<name>_tracks.csv` and `.json`, with the `_detections` or `_detect` suffix of the input removed.

---

## The tracks CSV

A row begins with **YORU's own detection columns, in YORU's order**, so code that reads YORU's `*_detect.csv` by position reads these rows too. The tracking columns follow:

```text
x1 y1 x2 y2 confidence class class_name total_time cx cy w h angle
frame_id track_id track_state predicted vx vy track_confidence association_cost
```

| Column | Meaning |
|---|---|
| `x1`, `y1`, `x2`, `y2` | Upright box (pixels): the detection's, or, for a predicted row, around the predicted box. |
| `confidence` | The detector's confidence. Empty in predicted rows. |
| `class`, `class_name` | Class of the latest detection matched to the track. |
| `total_time` | The frame's time, in seconds. In a live recording, seconds since the run started. |
| `cx`, `cy`, `w`, `h` | The box's centre and size (pixels). For oriented-box models, the rotated box. |
| `angle` | The box's rotation, in radians; `0` for upright boxes. An axis, not a heading: head and tail are not told apart. |
| `frame_id` | Frame number, from 0 in a video (Video Tracking shows `frame_id` 0 as *Frame 1*). In a live recording, the camera frame count, so a gap is a dropped frame. |
| `track_id` | The animal's ID: `0`, `1`, `2`, ... |
| `track_state` | `active` (detected in this frame) or `lost` (predicted). |
| `predicted` | `1` for a lost track's predicted position, otherwise `0`. |
| `vx`, `vy` | Velocity, in pixels per frame. |
| `track_confidence` | The detector's confidence smoothed over time, lowered while the track is unseen. |
| `association_cost` | Cost of this frame's match, for debugging; empty for a track's first row and for predicted rows. |

**Predicted rows** — where a lost track is estimated to be — are written only when asked for (*Include predicted rows of lost tracks*, *incl. predicted*, or `--include-predicted`). Without them, each row is a real detection, and a missing (`frame_id`, `track_id`) pair is a frame in which that animal was not detected.

**Frame rate.** `total_time` uses the video's frame rate. A video that states no frame rate has it measured from its frames' timestamps, or, without those, 30 fps assumed. The window, the command line and `source.fps_source` in the JSON (`file`, `timestamps` or `assumed`) say which.

### Reading it

With pandas, for example:

```python
import json
import pandas as pd

tracks = pd.read_csv("results/movie_tracks.csv")
meta = json.load(open("results/movie_tracks.json", encoding="utf-8"))
fps = meta["source"]["fps"]

# One animal's trajectory
fly0 = tracks[tracks.track_id == 0].sort_values("frame_id")

# Its speed in pixels per second
speed = (fly0.vx ** 2 + fly0.vy ** 2) ** 0.5 * fps

# x position of every animal, one column per ID
x = tracks.pivot(index="frame_id", columns="track_id", values="cx")
```

---

## The JSON file

Beside every tracks CSV, a JSON file records how it was made:

| Key | Content |
|---|---|
| `metadata_version`, `created_at` | Format version, and when the file was written. |
| `software` | Versions of YORU Tracker, YORU and Python, and the platform. |
| `tracker` | Tracker name, version, tracker API version and capabilities. |
| `tracker_config` | **The full tracker configuration**, in the settings-file format. |
| `detector` | Model path, backend, confidence and NMS IoU thresholds, excluded classes. |
| `source` | The input: path, frame rate and where it came from (`fps_source`), frame count, size. |
| `outputs` | The files written, the number of rows, whether predicted rows are included. |
| `summary` | Frames, whether the run was complete, number of track IDs, detector and tracker time per frame. |
| `track_columns` | The CSV's columns. |

With the detections CSV and this file, a run can be repeated exactly, on another machine and without a GPU. To get the configuration back in Python:

```python
from yoru_tracker.export import config_from_metadata, read_metadata

config = config_from_metadata(read_metadata("results/movie_tracks.json"))
config.save("movie_settings.yaml")      # usable with --config and Load...
```

---

## The detections CSV

`movie_detections.csv` holds what the detector found, frame by frame: `frame_id` followed by YORU's detection columns (`total_time` holding the frame's time). A frame in which nothing was detected is a row with only `frame_id` and `total_time` filled in, so the tracker sees exactly the frames it saw before.

Track it again with new settings:

- in the window: open the video in Video Tracking, then **Load detections CSV...**;
- on the command line: `yoru-tracker track results/movie_detections.csv --config new.yaml`.

### YORU's own detection files

The same two routes read YORU's files, so experiments recorded with YORU can be tracked afterwards:

| File | Notes |
|---|---|
| YORU's video-analysis CSV | Every frame between the first and last row was analysed; frames missing from the table are fed to the tracker as empty frames. |
| YORU's real-time `*_detect.csv` | A detection repeated over several video frames is counted once. With the matching `*_log.csv` beside it, frame IDs are the recorded video's frame numbers (required in Video Tracking). Frames in which the detector found nothing cannot be recovered from this file. |

Files saved from a spreadsheet often start with a byte-order mark; it is skipped.
