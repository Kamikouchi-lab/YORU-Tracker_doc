---
layout: page
title: Q and A
order: 2
---

> When you report a problem on [GitHub Issues](https://github.com/Kamikouchi-lab/YORU-Tracker/issues), please attach the log file `~/.yoru/logs/yoru_tracker.log` (`%USERPROFILE%\.yoru\logs\yoru_tracker.log` on Windows; **Help → Open log folder** opens it). It holds the full traceback of every error shown in the window.
>
> Problems with the detector itself — CUDA, GPU memory, loading a model — are YORU's: see YORU's [Q and A]({{ site.yoru.docs }}troubleshootings/).

***
## Installation and startup

**Q. `uv sync` fails because it cannot find or accept `yoru`.**

**A.** YORU Tracker installs YORU from the folder `../YORU`, and needs YORU 2.0.0 Beta 4 or later. Check that the `YORU` folder is beside `YORU-Tracker` (not inside it), and that it is at that release: `git -C ../YORU fetch --tags`, `git -C ../YORU checkout v2.0.0-beta.4`, then `uv sync` again. The stable YORU v1.1.x does not work. See [Install]({{ site.baseurl }}/guides/01-install/).

**Q. `python -m yoru_tracker` says Python was not found, or opens the Microsoft Store.**

**A.** On Windows, a bare `python` may be the Microsoft Store stub. Run YORU Tracker with `uv run yoru-tracker` from the `YORU-Tracker` folder, or with `.venv\Scripts\python.exe -m yoru_tracker`.

**Q. Something failed, but the message in the window was too short.**

**A.** Every error shown in the window is written with its traceback to `~/.yoru/logs/yoru_tracker.log`. **Help → Open log folder** opens the folder.

**Q. The window opens with strange settings or folders from an earlier session.**

**A.** They are remembered in `~/.yoru/yoru_tracker_gui.json`. Press **Defaults** then **Apply** in the settings window, or delete the file to start fresh.

## Tracking results

**Q. I have two animals, but the result has many more track IDs.**

**A.** Each extra ID is a place where a track was lost and a new one started — usually when the animals touch and the detector sees one box. If the number of animals never changes, set **Number of animals** (`population`): no more IDs are then given out, and an animal that reappears gets its own ID back. Then check **Max distance** and **Max age**. See [Tracker Settings]({{ site.baseurl }}/guides/05-tracker-settings/#which-settings-to-change).

**Q. Two animals swap IDs after they have been in contact.**

**A.** While the detector reports two touching animals as one box, which animal leaves the box on which side cannot be told from position alone. Setting `population` reduces this a lot, but cannot always prevent it. Appearance-based re-identification is the job of the planned Advanced tracker. See [How the Lite Tracker Works]({{ site.baseurl }}/guides/08-lite-tracker/#what-lite-cannot-do-yet).

**Q. An animal gets a new ID whenever its behaviour class changes (e.g. `solo` → `copulation`).**

**A.** **Match within class only** (`class_aware`) is on. Turn it off when classes are behaviours: an animal changes behaviour, not identity. Keep it on only when classes are different animals.

**Q. A box around two touching animals gets an ID of its own.**

**A.** Give the number of animals (`population`) if it is known. Otherwise raise **Trusted confidence** (`high_confidence`) above the confidence your detector gives such boxes, and keep **Keep hidden animals' IDs in place** (`hidden_guard`) on. See [Crowded arenas]({{ site.baseurl }}/guides/08-lite-tracker/#crowded-arenas-spare-boxes).

**Q. I changed the settings, but the result did not change.**

**A.** Press **Apply** in the settings window first. Then: in Video Tracking press **Re-track**; in Realtime Tracking press **Reset tracker** (IDs restart at 0); in Batch Tracking start the batch again.

**Q. A lost animal has no rows in the tracks CSV.**

**A.** Only real detections are written by default. To get the predicted positions of lost tracks too, tick *Include predicted rows of lost tracks* (or use `--include-predicted`); those rows have `predicted` = 1.

**Q. The times in seconds look wrong, and the status says *30 fps assumed, so times in seconds are a guess*.**

**A.** The video file states no frame rate and has no frame timestamps, so 30 fps was assumed for `total_time`. Frame numbers (`frame_id`) are still right; convert them with the frame rate you recorded at. `source.fps_source` in the JSON says where the frame rate came from.

## Settings files

**Q. Loading a settings file fails with `unknown key ...` or `... must be within [0, 1]`.**

**A.** Loading is strict on purpose: a misspelt or out-of-range setting is never silently replaced by a default. Fix every key the message lists. `uv run yoru-tracker config` prints a valid file to start from. See [Loading is strict]({{ site.baseurl }}/guides/05-tracker-settings/#loading-is-strict).

**Q. Choosing the mode `advanced` gives an error.**

**A.** The Advanced tracker is planned but not part of this release. Use `lite`. YORU Tracker never falls back from one mode to another silently.

**Q. Realtime Tracking says *Tracker cannot run live*.**

**A.** The tracker mode chosen in the settings is not realtime-capable. Choose `lite`.

## Files and inputs

**Q. `--render needs the video itself; a detections CSV has no frames to draw on`.**

**A.** A detections CSV has no images. Track the video with `--model ... --render`, or open the video in Video Tracking, **Load detections CSV...**, then **Render video**.

**Q. `A video needs --model (or give a detections CSV instead).`**

**A.** Tracking a video runs the detector: give the model with `--model best.pt`. To track without a model, give the detections CSV saved from an earlier run.

**Q. *... is a YORU real-time detections file without its \*\_log.csv beside it*.**

**A.** Video Tracking needs YORU's `<name>_log.csv` in the same folder as `<name>_detect.csv`: only the log says which video frame each detection belongs to. Copy it there and load again.

**Q. *... has detections up to frame N, but the video has M frames: it belongs to another video*.**

**A.** The detections CSV was made from a different (longer) video than the one open. Open the video it belongs to.

**Q. *not a detections file YORU Tracker can read*.**

**A.** YORU Tracker reads its own `*_detections.csv`, YORU's video-analysis CSV and YORU's real-time `*_detect.csv`, told apart by their header. The file has another header — for example, it is a `*_tracks.csv` or a `*_log.csv`.

**Q. My results disappeared when I ticked *Flip vertical* / *Flip horizontal*.**

**A.** A flip reopens the video; results computed on the unflipped frames no longer match them and are cleared. Set the flips before **Run tracking**.

**Q. *The current video is still in use*.**

**A.** A run, a re-track, an export or a render is still going on. Wait for it to finish, or press **Stop**, before opening another video.

## Realtime Tracking

**Q. *Camera N returned no frame; check the connection*.**

**A.** Check the **Camera ID** (`0` is the first camera), the cable, and that no other program (for example, YORU itself) is using the camera.

**Q. *Dropped frames* keeps growing.**

**A.** The detector is slower than the camera, so frames in between are skipped rather than queued — the display stays current, and the tracker predicts motion over the longer step. To drop fewer frames, use a smaller or faster model, or a GPU. Compare *Camera FPS* with *Detector FPS*.

**Q. My live recording was split into `..._part2_tracks.csv`, `..._part3_tracks.csv`.**

**A.** The tracker was reset during the recording. IDs restart at 0 after a reset, so the recording continues in a new file: no file holds two animals under one ID. Each part has its own JSON.

## Batch Tracking

**Q. A file failed. Did the batch stop?**

**A.** No. A failed file is marked `failed` with its error, and the batch moves on. The **Failure report** (or, on the command line, the report at the end) lists every failure; the tracebacks are in the log.

**Q. `No videos found in ...` on the command line.**

**A.** Only `.mp4`, `.avi`, `.mov`, `.mkv`, `.wmv`, `.m4v`, `.mpg` and `.mpeg` files are tracked. Videos in subfolders need `--recursive` (*Include subfolders* in the window).

## Python API

**Q. `frame_id N does not follow M: frames must be given in increasing order`.**

**A.** `update` must be called once per frame, with increasing `frame_id`. To start a new video with the same tracker, call `tracker.reset()` first — or create a new tracker. See [Tracker API]({{ site.baseurl }}/devnotes/01-tracker-api/#the-contract).
