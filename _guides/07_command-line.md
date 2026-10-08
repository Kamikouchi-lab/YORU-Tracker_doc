---
layout: page
title: Command Line
order: 7
---

Everything the window does for videos and folders can be run without it. Run the commands from the `YORU-Tracker` folder with `uv run`:

```
uv run yoru-tracker <command> [options]
```

| Command | Does |
|---|---|
| *(none)* or `gui` | Open the window. |
| [`track`](#track) | Track one video, or re-track a saved detections CSV. |
| [`batch`](#batch) | Track every video in a folder. |
| [`bench`](#bench) | Score trackers on the synthetic scenarios. |
| [`config`](#config) | Print the default tracker settings. |

`uv run yoru-tracker --help` and `uv run yoru-tracker <command> --help` list the options; `--version` prints the version. Help answers at once, without loading PyTorch.

---

## gui

```
uv run yoru-tracker
uv run yoru-tracker gui --view video
```

| Option | |
|---|---|
| `--view {home,video,realtime,batch}` | Open this screen instead of the start screen. |

---

## track

Track one video:

```
uv run yoru-tracker track movie.mp4 --model best.pt --out results/
uv run yoru-tracker track movie.mp4 --model best.pt --out results/ --render
uv run yoru-tracker track movie.mp4 --model best.pt --config two_flies.yaml --include-predicted
```

Re-track saved detections, without a model or GPU:

```
uv run yoru-tracker track results/movie_detections.csv --config two_flies.yaml
uv run yoru-tracker track recordings/session1_detect.csv --out results/
```

The input is a video, or a CSV: YORU Tracker's `*_detections.csv`, YORU's video-analysis CSV, or YORU's real-time `*_detect.csv` (with its `*_log.csv` beside it, frame IDs are the recorded video's frame numbers). See [Output Files]({{ site.baseurl }}/guides/06-output-files/#the-detections-csv).

| Option | Default | |
|---|---|---|
| `input` | | A video file, or a detections CSV. |
| `--out DIR` | beside the input | Output folder. |
| `--render` | off | Also write `<video>_tracked.mp4`. Needs a video, not a CSV. |
| `--model PATH` | | Detector model (any YORU backend). Required for a video. |
| `--backend NAME` | `auto` | Detector backend. |
| `--conf X` | `0.25` | Detector confidence threshold. |
| `--iou X` | `0.45` | Detector NMS IoU threshold. |
| `--exclude-class ID` | | A class ID not to track. Repeat it for several classes. |
| `--config FILE` | defaults | Tracker settings (YAML or JSON). |
| `--mode NAME` | from the settings | Override the tracker mode (`lite`, `baseline`, ...). |
| `--include-predicted` | off | Also write rows for lost tracks' predicted positions. |

A video run prints its progress every 10 %, then a summary and the files written, for example:

```text
    0% (1/9002 frames)
   10% (901/9002 frames)
  ...
9002 frames, 2 track IDs; detector 8.4 ms/frame, tracker 0.180 ms/frame
  tracks_csv: results/movie_tracks.csv
  detections_csv: results/movie_detections.csv
  metadata: results/movie_tracks.json
```

If the video states no frame rate, a note on standard error says how it was obtained.

---

## batch

```
uv run yoru-tracker batch videos/ --model best.pt --out results/
uv run yoru-tracker batch videos/ --model best.pt --out results/ --recursive --config two_flies.yaml
```

| Option | Default | |
|---|---|---|
| `folder` | | The folder of videos. |
| `--out DIR` | | Output folder. **Required.** |
| `--recursive` | off | Include subfolders; the output folder mirrors them. |
| `--model PATH` | | Detector model. **Required.** |
| `--backend`, `--conf`, `--iou`, `--exclude-class` | | As for `track`. |
| `--config`, `--mode`, `--include-predicted` | | As for `track`. |

One line is printed per finished file, then the failure report:

```text
[   done] day1/fly.mp4  frames=9002 ids=2
[ failed] day2/broken.avi  frames=0 ids=0  OSError: ...
1 file(s) failed:
- videos/day2/broken.avi: OSError: ...
```

Outputs are named as in [Batch Tracking]({{ site.baseurl }}/guides/04-batch-tracking/#output-files): subfolders are mirrored and no result overwrites another. The batch exits with code 1 if any file failed.

---

## bench

```
uv run yoru-tracker bench
uv run yoru-tracker bench --config my_settings.yaml
uv run yoru-tracker bench --scenario contact --scenario jump --tracker lite
```

| Option | Default | |
|---|---|---|
| `--scenario NAME` | all | A scenario to run. Repeatable. |
| `--tracker MODE` | `lite` and `baseline` | A tracker mode to score. Repeatable. |
| `--config FILE` | | Also score these settings, as `custom`. |
| `--seed N` | `0` | Seed of the synthetic scenarios. |
| `--json PATH` | | Also write the rows as JSON. |

See [Benchmark]({{ site.baseurl }}/guides/09-benchmark/) for the scenarios and the columns.

---

## config

```
uv run yoru-tracker config > tracker.yaml
```

Prints the default tracker settings as YAML. Edit the file and pass it with `--config`, or load it in the settings window. See [The settings file]({{ site.baseurl }}/guides/05-tracker-settings/#the-settings-file).

---

## Exit codes and errors

| Code | |
|---|---|
| `0` | Success. |
| `1` | An error, or a batch in which a file failed. Errors are printed on standard error — as `[yoru-tracker] <ErrorType>: <message>` with the traceback in `~/.yoru/logs/yoru_tracker.log`, or, for a wrong combination of arguments, as a plain message saying what to do. |
| `130` | Interrupted with <kbd>Ctrl</kbd>+<kbd>C</kbd>. |
