---
layout: page
order: 3
title: Step3. Tune and Re-track
---

The detections of Step 2 are kept, so each change of settings is applied to the whole video in seconds with **Re-track**.

1. Press **Settings...**.

2. Set **Number of animals (0 = ?)** to `2` and press **Apply**.

    > A note appears below *Re-track*: the settings changed, and the video has not been tracked with them yet.

3. Press **Re-track**. The status line now reads *Re-tracked N frames with mode lite: K track IDs*.

    > With the number of animals known, no more than 2 IDs are ever given out, and no ID is retired. A fly that reappears after its partner covered it gets its own ID back.

4. Check the result as in Step 2, especially around contacts. If needed, change one more setting at a time, **Apply** and **Re-track**:

    | What you see | Try |
    |---|---|
    | A fly's ID is lost when it walks fast | Raise **Max distance (px)** |
    | An ID jumps to the other fly | Lower **Max distance (px)** |
    | The detector misses a fly for many frames | Raise **Max age (frames)** |
    | Classes are animals (e.g. `male` / `female`) | Tick **Match within class only** |

    The [Tracker Settings]({{ site.baseurl }}/guides/05-tracker-settings/) guide explains every setting.

5. Check the settings on a second video (`pair02.mp4`): **Run tracking**, and look again. Settings that only fit one video are a trap.

6. In the settings window, press **Save...** and save the settings, for example as `D:\experiment\two_flies.yaml`.

7. **Export** this video if you want it now: choose an **Output folder**, keep *Save detections* ticked, and press **Export CSV**. **Render video** writes the video with IDs and trails drawn on it — useful for checking the result, or for a presentation.

The saved file holds only settings, for example:

```yaml
config_version: 2
tracker:
  mode: lite
  lifecycle:
    max_age: 10
    min_hits: 2
    population: 2
  ...
```
