---
layout: page
title: Video Tracking
order: 2
---

Video Tracking runs one video through the detector and the tracker, then lets you inspect every frame: its tracks, their state and match cost, and what happened in that frame. It is also where the tracker settings are best tuned, because a changed setting is applied to the whole video in seconds with **Re-track**.

Open it with **Video Tracking** on the start screen, or **Go → Video Tracking**.

---

## The screen

<div class="screen-map video">
  <div class="controls">
    <strong><span class="num">1</span>Input</strong>
    Video · Detector model · Backend · Conf. · Flip vertical / horizontal
    <br><br>
    <strong><span class="num">2</span>Tracking</strong>
    Settings... · Run tracking · Stop · progress · Re-track · Load detections CSV...
    <br><br>
    <strong><span class="num">3</span>Export</strong>
    Output folder · Include predicted rows · Save detections · Export CSV · Render video
  </div>
  <div><strong><span class="num">4</span>Image</strong>the frame, with the overlays switched on below</div>
  <div><strong><span class="num">5</span>Timeline</strong>slider · |&lt; &lt; Play &gt; &gt;| · frame, time, active / lost count</div>
  <div><strong><span class="num">6</span>Overlays</strong>Track boxes · IDs · Trails · Detections · Predicted (lost) · Velocity · Confidence</div>
  <div><strong><span class="num">7</span>Inspection</strong>Tracks in this frame · Events in this frame</div>
</div>

---

## Track a video

1. **Video** — press *Browse* and choose the video (`.mp4`, `.avi`, `.mov`, `.mkv`, `.wmv`, `.m4v`, `.mpg`, `.mpeg`).

    The status line shows the number of frames, the size and the frame rate. If the file states no frame rate, YORU Tracker measures it from the frames' timestamps, or, without those, assumes 30 fps; the status line says which.

2. **Detector model** — press *Browse* and choose a model trained in YORU (`.pt`, `.pth` or `.onnx`).

3. **Backend** — leave it at `auto` to let YORU pick the detector backend from the model, or choose one from the list.

4. **Conf.** — the detector's confidence threshold (default `0.25`). Detections below it are never passed to the tracker.

5. **Flip vertical / Flip horizontal** — if the video has to be flipped. Changing a flip reopens the video and clears any results, since they no longer match the frames.

6. **Settings...** — check the [tracker settings]({{ site.baseurl }}/guides/05-tracker-settings/). The line beside the button shows the mode, `max_age` and `max_distance` in use. If you know how many animals are in view, set **Number of animals** now.

7. Press **Run tracking**.

    The view follows the run, and the status line shows the progress and the detector and tracker time per frame. Any frame already processed can be inspected while the run goes on. **Stop** ends the run early; the frames tracked so far can still be inspected and exported.

---

## Inspect the result

### Moving through the video

| Control | Key | Action |
|---|---|---|
| Slider | | Go to any frame. |
| `|<` / `>|` | <kbd>Home</kbd> / <kbd>End</kbd> | First / last frame. |
| `<` / `>` | <kbd>←</kbd> / <kbd>→</kbd> | One frame back / forward. |
| Play / Pause | <kbd>Space</kbd> | Play at the video's frame rate. |

The line beside the buttons shows the frame number, its time, and the number of active and lost tracks. On a frame the detector never ran on (for example in a real-time recording loaded with *Load detections CSV*), it shows the tracks of the latest frame before it, marked *(not detected; tracks of frame N)*.

### Overlays

| Overlay | Default | Draws |
|---|---|---|
| Track boxes | on | Each track's box, in the track's own colour. |
| IDs | on | The track ID. |
| Trails | on | The track's last 30 positions. |
| Detections | off | The detector's raw boxes, before tracking. |
| Predicted (lost) | off | Where a lost track is predicted to be. With it off, trails run only through where the animals were seen. |
| Velocity | off | An arrow for each track's velocity. |
| Confidence | off | Each track's confidence, next to its ID. |

The overlays change only the display, and what **Render video** draws. They do not change the tracks.

### Tracks in this frame

One row per track in the shown frame, in the track's colour (grey when it is only predicted):

| Column | Meaning |
|---|---|
| ID | Track ID. |
| state | `active`, or `lost` (not detected in this frame). |
| class | Class of the latest detection matched to the track. |
| conf | Track confidence: the detector's confidence smoothed over time, lowered while the track is unseen. |
| hits | How many times the track has been detected. |
| missed | Frames since it was last detected. |
| vx, vy | Velocity, in pixels per frame. |
| cost | Cost of this frame's match (lower is a better match). Empty for new or predicted tracks. |

### Events in this frame

What happened to tracks in the shown frame: `created` (a track got its ID), `lost`, `recovered` (a lost track was found again), `retired` (lost for longer than `max_age`; its ID is not used again).

Look for frames where an ID is created while another is lost: that is where identities may have been swapped or split. The [Lite tracker]({{ site.baseurl }}/guides/08-lite-tracker/) page explains why that happens and the [Tracker Settings]({{ site.baseurl }}/guides/05-tracker-settings/) page what to change.

---

## Re-track with new settings

Detection is the slow part; tracking takes a fraction of a millisecond per frame. YORU Tracker keeps the detections of the run, so:

1. Open **Settings...**, change the settings, and press **Apply**.

    A note appears: *Tracker settings changed: press Re-track to apply them to this video.*

2. Press **Re-track**. The stored detections are tracked again with the new settings, without running the model. The status line shows the number of track IDs and the tracking time.

Repeat until the IDs are right, then export. **Save...** in the settings window keeps the settings as a YAML file for Batch Tracking or the command line.

---

## Load a detections CSV

**Load detections CSV...** tracks detections saved earlier, laid over the open video, without running a model. Open the video they belong to first. Three kinds of file are read:

| File | Written by |
|---|---|
| `<video>_detections.csv` | YORU Tracker (*Export CSV* with *Save detections*, or the command line). |
| YORU's video-analysis CSV | YORU's video analysis. Frames missing from the table are frames with no detections. |
| `<name>_detect.csv` | YORU's real-time process. Its `<name>_log.csv` must be **in the same folder**: only the log says which video frame each detection belongs to. |

In a real-time recording, the detector did not run on every video frame. Frames it skipped are stepped over by the tracker, not counted as frames in which every animal was missed.

A file with detections beyond the last frame of the open video is refused: it belongs to another video.

---

## Export

1. **Output folder** — where the files go. Left empty, they go beside the video.
2. **Include predicted rows of lost tracks** — also write a row for each lost track's predicted position (off by default).
3. **Save detections (re-track later without the model)** — also write the detector's output (on by default).
4. Press **Export CSV**. For `movie.mp4` it writes:

    | File | Content |
    |---|---|
    | `movie_tracks.csv` | One row per track per frame. |
    | `movie_tracks.json` | Everything needed to reproduce the CSV. |
    | `movie_detections.csv` | The detector's output, if *Save detections* is on. |

5. Press **Render video** to write `movie_tracked.mp4`: the video with the overlays that are switched on now, and the same flips.

Existing files of the same name are overwritten. The columns are described in [Output Files]({{ site.baseurl }}/guides/06-output-files/).

<div class="note" markdown="1">
While a run, a re-track or an export is going on, another video cannot be opened and the flips cannot be changed. Wait for it to finish, or press **Stop**.
</div>
