---
layout: page
title: Overview
order: 0
---

These pages describe how to use **YORU Tracker {{ site.version }}**.

YORU finds animals and behaviours in each frame, but a detection does not say *which* animal it is. YORU Tracker links the detections of consecutive frames, so that each animal keeps one ID for the whole video — its **track**. From the tracks you get trajectories, speeds, and per-animal behaviour over time.

---

## Where YORU Tracker fits

```text
YORU                          YORU Tracker
----                          ------------
label frames                  camera / video / folder of videos
     |                                   |
train a model  --- best.pt --->  detector (your YORU model)
                                         |  detections, frame by frame
                                         v
                                 tracker (Lite)  --->  IDs, trails, events
                                         |
                                         v
                  *_tracks.csv   *_tracks.json   *_detections.csv
```

1. **Train a detector in YORU.** Labelling, training and evaluation are done in YORU; see the [YORU documentation]({{ site.yoru.docs }}). YORU Tracker has no detector of its own.
2. **Track with YORU Tracker.** Load the trained model (`.pt`, `.pth` or `.onnx`) and a camera, a video or a folder of videos.
3. **Analyse the tracks.** The tracks CSV has one row per animal per frame, starting with the same columns as YORU's detection files.

The normal YORU application is unchanged by YORU Tracker. The two are separate programs that share YORU's detectors.

---

## The three workflows

| Workflow | Input | Use it to |
|---|---|---|
| [Video Tracking]({{ site.baseurl }}/guides/02-video-tracking/) | One video, or a detections CSV laid over its video | Track a recording, check every frame, tune the settings, export and render. |
| [Realtime Tracking]({{ site.baseurl }}/guides/03-realtime-tracking/) | A camera, or a video played in real time | See IDs live, measure latency, record tracks as they come. |
| [Batch Tracking]({{ site.baseurl }}/guides/04-batch-tracking/) | A folder of videos | Track many recordings with one model and one set of settings. |

All three use the same tracker and the same [Tracker Settings]({{ site.baseurl }}/guides/05-tracker-settings/). The same detections give the same IDs whether they come from a camera, a video or a file.

A good way to start is **Video Tracking** on a short recording: run it once with the default settings, look at the frames where IDs change, adjust the settings, and press *Re-track*. When the settings work, use them in Batch or Realtime Tracking.

---

## The start screen and menus

`uv run yoru-tracker` opens the start screen. It has a button for each workflow and a **Settings** button. Its footer shows the versions of YORU Tracker, YORU and the tracker API, and the path of the log file.

The menu bar is available on every screen:

| Menu | Items |
|---|---|
| **Go** | Home, Realtime Tracking, Video Tracking, Batch Tracking |
| **Tracker** | Settings... (the [tracker settings]({{ site.baseurl }}/guides/05-tracker-settings/) window) |
| **Help** | Open log folder; the YORU Tracker and tracker API versions |
| **Window** | Fit window to this screen, Maximize window, Text size, Save layout now, Reset layout to default (the same menu as in YORU) |

You can also open one screen directly: `uv run yoru-tracker gui --view video` (or `realtime`, `batch`, `home`).

The detector model, the confidence threshold, the folders you last used, the camera ID and the tracker settings are remembered between sessions in `~/.yoru/yoru_tracker_gui.json`. A model selected on one screen is selected on all of them.

---

## Words used in these guides

| Term | Meaning |
|---|---|
| **Detection** | One box from the detector in one frame: position, size, angle (for oriented boxes), confidence and class. |
| **Track** | One animal followed over time. |
| **Track ID** | The animal's number: `0`, `1`, `2`, ... given in the order tracks are confirmed. IDs restart at 0 only when the tracker is reset. |
| **Tentative** | A newcomer that has not been seen often enough yet (see `min_hits`). It has no ID. |
| **Active** | A track matched to a detection in this frame. |
| **Lost** | A track not detected in this frame. Its position is predicted, and it keeps its ID for up to `max_age` frames. |
| **Retired** | A track lost for longer than `max_age`. Its ID is not used again. |
| **Predicted row** | A lost track's estimated position. Shown and exported only when you ask for it. |
| **Trail** | The track's last 30 positions, drawn behind it. |
| **Event** | Something that happened to a track in a frame: `created`, `lost`, `recovered`, `retired`, or `reset` (every track dropped). |
| **Tracker mode** | Which tracker runs: `lite` (the default) or `baseline` (YORU's frame-to-frame matching, for comparison). `advanced` is planned. |

---

## Two rules YORU Tracker keeps

- **Reproducible.** The same detections, frame IDs, timestamps and settings always give the same tracks. There is no randomness and no clock inside a result.
- **Never fails silently.** A misspelt setting is an error naming it, not a silent default. One tracker mode never falls back to another. A batch file that fails is reported with its error. Every error shown in the window is also written to `~/.yoru/logs/yoru_tracker.log`.
