---
layout: page
order: 2
title: Step2. Track a Video
---

First, track one video with the default settings, to see what the tracker does with your detector.

1. Launch YORU Tracker and press **Video Tracking**.

    ```
    uv run yoru-tracker
    ```

2. **Video** → *Browse* → `pair01.mp4`.

    > Check the status line: number of frames, size and frame rate. If it says the frame rate was measured or assumed, keep it in mind when you convert frames to seconds.

    > If the image is upside down or mirrored compared with how you labelled it in YORU, tick **Flip vertical** / **Flip horizontal**.

3. **Detector model** → *Browse* → your YORU model. Leave **Backend** at `auto` and **Conf.** at `0.25`.

4. Press **Settings...**, then **Defaults**, then **Apply** — so that this first run uses the default settings.

5. Press **Run tracking**.

    > The view follows the run. The status line shows how many frames are done, how many tracks there are now, and the detector and tracker time per frame. Detection takes most of the time; tracking a fraction of a millisecond per frame.

6. When the run is finished, read the status line: *Done: N frames, K track IDs.*

    With two flies and default settings, **K is often more than 2**. Each extra ID is a place where the tracker lost a fly and started a new track — most often while the two flies are in contact and the detector sees one box (for example, during copulation).

7. Find those places. Drag the slider, or step with <kbd>←</kbd> / <kbd>→</kbd> (<kbd>Space</kbd> plays), and watch:

    - **Events in this frame** — a `created` event after the first frames is a new ID; a `lost` event is a fly the detector missed.
    - **Tracks in this frame** — `missed` counts the frames a track has gone undetected; `cost` is high for a doubtful match.
    - Tick **Detections** under the image to see the detector's own boxes beside the tracks.

You now know where the IDs break. In the next step, the settings fix most of it — without running the detector again.
