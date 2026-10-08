---
layout: page
title: Batch Tracking
order: 4
---

Batch Tracking tracks every video in a folder with one model and one set of tracker settings, and writes each video's results to an output folder. The model is loaded once. Each video gets a fresh tracker, so IDs never carry over from one file to the next.

Open it with **Batch Tracking** on the start screen, or **Go → Batch Tracking**.

<div class="note" markdown="1">
Tune the settings on one or two videos in [Video Tracking]({{ site.baseurl }}/guides/02-video-tracking/) first, then run the batch with them.
</div>

---

## The screen

<div class="screen-map batch">
  <div class="controls">
    <strong><span class="num">1</span>Input</strong>
    Video folder · Include subfolders · Detector model · Conf.
    <br><br>
    <strong><span class="num">2</span>Tracking</strong>
    Settings... · Output folder · Include predicted rows · Run batch · Stop · progress
  </div>
  <div><strong><span class="num">3</span>Files</strong>File · Status · Frames · Track IDs · Message — one row per video</div>
  <div><strong><span class="num">4</span>Failure report</strong>every file that failed, with its error</div>
</div>

---

## Run a batch

1. **Video folder** — press *Browse* and choose the folder. Every video in it (`.mp4`, `.avi`, `.mov`, `.mkv`, `.wmv`, `.m4v`, `.mpg`, `.mpeg`) is listed in the *Files* table.
2. **Include subfolders** — also track the videos in its subfolders.
3. **Detector model** — press *Browse* and choose a model trained in YORU. **Conf.** is the detector's confidence threshold (default `0.25`).
4. **Settings...** — check the [tracker settings]({{ site.baseurl }}/guides/05-tracker-settings/), or **Load...** the YAML file you saved while tuning, then **Apply**.
5. **Output folder** — press *Browse* and choose where the results go. It is required.
6. **Include predicted rows of lost tracks** — also write rows for lost tracks' predicted positions (off by default).
7. Press **Run batch**. The progress bar and the status line follow the batch.

**Stop** stops after the current frame. The file being tracked is saved with the frames done so far and marked `stopped`; the files after it are not started and stay `pending`.

---

## The Files table

| Column | Meaning |
|---|---|
| File | The video's path below the chosen folder. |
| Status | `pending`, `running`, `done`, `stopped` or `failed`. |
| Frames | Frames processed, out of the total. |
| Track IDs | The number of track IDs in the video, once it is `done`. |
| Message | The error of a failed file, or a note (for example, that its frame rate had to be guessed). |

A file that fails is marked `failed` with its error, and **the batch moves on** to the next one. When the batch ends, the **Failure report** lists every failed file with its error. Failures are also written to `~/.yoru/logs/yoru_tracker.log`.

---

## Output files

Each video writes the same files as [Export CSV]({{ site.baseurl }}/guides/02-video-tracking/#export) in Video Tracking: `<video>_tracks.csv`, `<video>_tracks.json` and `<video>_detections.csv`. Batch Tracking does not render videos; render the ones you want to check from Video Tracking, or with `--render` on the [command line]({{ site.baseurl }}/guides/07-command-line/).

The names are chosen so that **no result overwrites another**:

- **Subfolders are mirrored.** With *Include subfolders*, `day1/fly.mp4` and `day2/fly.mp4` write `day1/fly_tracks.csv` and `day2/fly_tracks.csv` in the output folder.
- **Same name, different extension.** Videos in one folder that differ only in their extension get it added: `fly.avi` and `fly.mp4` write `fly_avi_tracks.csv` and `fly_mp4_tracks.csv`.

```text
videos/                         results/
├── day1/                       ├── day1/
│   ├── fly.mp4        ──▶      │   ├── fly_tracks.csv, fly_tracks.json, fly_detections.csv
│   └── pair.avi       ──▶      │   └── pair_tracks.csv, ...
└── day2/                       └── day2/
    ├── fly.avi        ──▶          ├── fly_avi_tracks.csv, ...
    └── fly.mp4        ──▶          └── fly_mp4_tracks.csv, ...
```

The same batch can be run headless with `yoru-tracker batch`; see [Command Line]({{ site.baseurl }}/guides/07-command-line/#batch).
