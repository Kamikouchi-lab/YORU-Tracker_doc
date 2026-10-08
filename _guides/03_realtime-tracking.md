---
layout: page
title: Realtime Tracking
order: 3
---

Realtime Tracking runs camera, detector and tracker live. The frame with the detector's boxes is shown beside the same frame with tracks, IDs and trails, and below them the tracker's status, latency and events. The tracks can be recorded to CSV as they come.

Open it with **Realtime Tracking** on the start screen, or **Go → Realtime Tracking**.

---

## The screen

<div class="screen-map realtime">
  <div class="wide"><strong><span class="num">1</span>Source and model</strong>Source: Camera / Video file (real time) · Camera ID · video Browse · Model · Conf. · Tracker · Settings...</div>
  <div class="wide"><strong><span class="num">2</span>Run and record</strong>Start · Stop · Reset tracker · Record tracks to [folder] · Start recording · incl. predicted</div>
  <div><strong><span class="num">3</span>Raw / detection</strong>the frame with the detector's boxes</div>
  <div><strong><span class="num">4</span>Tracking</strong>the same frame with tracks, IDs and trails</div>
  <div class="wide"><strong><span class="num">5</span>Overlays</strong>Track boxes · IDs · Trails · Predicted (lost) · Velocity · Confidence</div>
  <div class="wide row3">
    <div><strong><span class="num">6</span>Tracking status</strong>IDs, frame rates, latency, drops</div>
    <div><strong><span class="num">7</span>Tracks</strong>one row per track</div>
    <div><strong><span class="num">8</span>Events</strong>newest first</div>
  </div>
</div>

---

## Start live tracking

1. **Source**

    - **Camera** — enter the **Camera ID** (`0` for the first camera).
    - **Video file (real time)** — press *Browse* and choose a video. It is played at its own frame rate, as if it came from a camera, and starts again from the beginning when it ends, so you can try settings without an animal under the lens. Frames the detector is too slow for are dropped, as with a camera.

2. **Model** — press *Browse* and choose a model trained in YORU. **Conf.** is the detector's confidence threshold (default `0.25`).

3. **Settings...** — check the [tracker settings]({{ site.baseurl }}/guides/05-tracker-settings/). The tracker mode must be one that can run live; `lite` can.

4. Press **Start**. The message line shows *Starting: ..., loading the model...*, then *Running: ...*.

5. Press **Stop** to end the run.

---

## Reading the screen

### Tracking status

| Field | Meaning |
|---|---|
| Tracker mode | The tracker running now. |
| Active IDs / Lost IDs | IDs matched in the latest frame / IDs not detected but kept. |
| Camera FPS | Frames per second delivered by the source. |
| Detector FPS | Frames per second the detector and tracker processed. |
| Detector latency | Time to detect one frame. |
| Tracking latency | Time for one tracker update (well under a millisecond with Lite). |
| Queue wait | From capture until the detector picked the frame up. |
| Draw time | Time to draw the frame on screen. |
| Capture to screen | From capture until the frame was shown. |
| Dropped frames | Frames skipped because the detector was busy, out of all frames captured. |
| Tracker resets | How many times the tracker was reset in this run. |
| Recording | Rows written so far, or `off`. |

When the detector is slower than the camera, the frames in between are **dropped and counted, not queued**, so the display never falls further and further behind. The tracker knows how many frames were skipped and predicts the animals' motion over the longer step.

### Tracks and Events

The **Tracks** table has the same columns as in [Video Tracking]({{ site.baseurl }}/guides/02-video-tracking/#tracks-in-this-frame). The **Events** log shows the latest `created`, `lost`, `recovered`, `retired` and `reset` events, newest first.

### Overlays

The overlays of the *Tracking* image are switched on and off below the images (the *Raw / detection* image always shows the detector's boxes). They change only the display, not the tracks or the recording.

---

## Changing settings during a run

Tracker settings changed while a run is going do **not** reach it at once. The message line says: *Tracker settings changed. They apply when you press Reset tracker (IDs restart) or start again.*

**Reset tracker** drops every track, restarts the IDs at 0, and uses the current settings. It is the only moment new settings reach a live run.

---

## Record tracks

1. **Record tracks to** — press *Browse* and choose a folder.
2. **incl. predicted** — also write rows for lost tracks' predicted positions (off by default).
3. While the run is going, press **Start recording**. The tracks are written as they come, to

    ```text
    live_YYYYMMDD_HHMMSS_tracks.csv
    ```

    named after the time recording started.

4. Press **Stop recording** (or **Stop**) to close the file. A `live_YYYYMMDD_HHMMSS_tracks.json` with the settings and a summary is written beside it.

**Reset while recording.** After a reset the IDs restart at 0, so the recording continues in a **new file** — `live_..._part2_tracks.csv`, then `_part3_`, and so on, each with its own JSON. No file ever holds two animals under one ID.

In a live recording, `frame_id` counts the camera's frames, so a gap in `frame_id` is a dropped frame, and `total_time` is the time in seconds since the run started. The columns are described in [Output Files]({{ site.baseurl }}/guides/06-output-files/).

<div class="note" markdown="1">
Realtime Tracking records the **tracks**, not the video. To keep the video as well, record it with YORU's real-time process and track it afterwards in [Video Tracking]({{ site.baseurl }}/guides/02-video-tracking/#load-a-detections-csv) or with the [command line]({{ site.baseurl }}/guides/07-command-line/).
</div>
