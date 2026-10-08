---
layout: page
order: 5
title: Step5. Track YORU Recordings
---

Experiments already recorded with YORU — offline video analyses, or closed-loop sessions with YORU's real-time process — can be tracked afterwards, from YORU's own detection files. The detector does not run again.

---

## From YORU's video analysis

YORU's video analysis writes one CSV per video. Track it directly:

```
uv run yoru-tracker track D:\yoru_results\pair01.csv --config two_flies.yaml --out D:\experiment\results
```

Or, to see the result frame by frame, open the video in **Video Tracking** and press **Load detections CSV...**.

---

## From YORU's real-time process

A real-time session leaves, among other files, the recorded video, `<name>_detect.csv` (the detections) and `<name>_log.csv` (which video frame each detection belongs to). **Keep the `_log.csv` in the same folder as the `_detect.csv`.**

### In the window

1. Press **Video Tracking**.
2. **Video** → the session's recorded video.
3. Press **Settings...**, set the settings (for example, **Load...** `two_flies.yaml`), and **Apply**.
4. Press **Load detections CSV...** → `<name>_detect.csv`.

    > The detections are laid over the video, frame by frame, using the log. During the session the detector did not run on every video frame: frames it skipped are stepped over, not counted as frames in which every fly was missed. On such a frame, the frame label says *(not detected; tracks of frame N)*.

5. Check the tracks, then **Export CSV** (and **Render video** if you want).

### On the command line

```
uv run yoru-tracker track D:\session\fly_detect.csv --config two_flies.yaml --out D:\experiment\results
```

This writes `fly_tracks.csv` and `fly_tracks.json`.

<div class="warning" markdown="1">
YORU's real-time `*_detect.csv` has no row for a frame in which the detector found nothing, so such frames cannot be told apart from frames it skipped. Tracks of recordings in which animals are often undetected are better made from the video itself: **Run tracking** in Video Tracking.
</div>
