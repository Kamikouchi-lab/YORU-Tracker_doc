---
layout: default
title: Home
---

## YORU Tracker

<div class="badges" markdown="1">
[![Version](https://img.shields.io/badge/version-0.1.0-0e274c.svg)](https://github.com/Kamikouchi-lab/YORU-Tracker/blob/main/CHANGELOG.md)
[![Requires YORU](https://img.shields.io/badge/requires-YORU%20%E2%89%A5%202.0.0b4-f2d851.svg)](https://github.com/Kamikouchi-lab/YORU)
[![Python 3.10](https://img.shields.io/badge/python-3.10-1d4b8f.svg)](https://www.python.org/downloads/release/python-3100/)
[![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Documentation](https://img.shields.io/badge/docs-YORU%20Tracker-0e274c.svg)](https://kamikouchi-lab.github.io/YORU-Tracker_doc/)
[![Sponsor](https://img.shields.io/badge/Sponsor-%E2%9D%A4-ff69b4?logo=githubsponsors&logoColor=white)](https://github.com/sponsors/HMYamano)
[![GitHub stars](https://img.shields.io/github/stars/Kamikouchi-lab/YORU-Tracker.svg?style=social&label=Star)](https://github.com/Kamikouchi-lab/YORU-Tracker)
</div>

<img src="logos/YORU_Tracker_logo.png" width="60%" alt="YORU Tracker logo">

**YORU Tracker** is multi-animal tracking for [YORU](https://github.com/Kamikouchi-lab/YORU). YORU detects animals and behaviours in every frame; YORU Tracker gives each animal a **persistent ID across frames**, draws its **trajectory**, and **exports per-animal tracks** — live from a camera, from a video, or for a whole folder of videos.

It is a sister application, not a YORU plugin: it has its own window and its own command, and the normal YORU application is unchanged by it. It uses YORU's detectors through YORU's public API, so every model YORU can load — YOLOv5 checkpoints from YORU v1, YOLOv8 / YOLO11, RT-DETR, torchvision, ONNX, oriented boxes included — works here without conversion.

| | YORU | YORU Tracker |
|---|---|---|
| Purpose | Real-time detection, closed-loop triggers, recording | Identity, trajectories, tracking export |
| Launch | `python -m yoru` | `python -m yoru_tracker` (or `yoru-tracker`) |
| Depends on | — | YORU (≥ 2.0.0b4, < 3) |
| Documentation | [YORU documentation]({{ site.yoru.docs }}) | This site |

<div class="sponsor-cta">
  <p><strong>Support YORU.</strong> YORU and YORU Tracker are free, open-source research software. If they help your work, please consider sponsoring their development on GitHub Sponsors.</p>
  {% include sponsor-button.html %}
</div>


## Features

- **Persistent identities.** Every animal keeps one ID from frame to frame, through crossings, contacts and short detector misses.
- **Three workflows in one window.** *Realtime Tracking* (camera), *Video Tracking* (one video, frame by frame) and *Batch Tracking* (a whole folder).
- **Any YORU model.** Detection goes through YORU's detector registry, so the models you trained in YORU work as they are.
- **Re-track in seconds.** The detector's output is saved; after changing the tracker settings, a video is tracked again without running the model or a GPU.
- **Track YORU's own recordings.** YORU's video-analysis CSV and real-time `*_detect.csv` can be tracked afterwards.
- **Known number of animals.** Tell the tracker how many animals are in view (for example, a pair of flies in a chamber) and it never invents or retires an ID.
- **Reproducible.** The same detections and settings always give the same IDs, and every tracks CSV comes with a JSON file that records everything needed to reproduce it.
- **Light enough for closed loops.** The *Lite* tracker runs no neural network of its own and takes about 0.1–0.5 ms per frame for a few animals.
- **Command line.** Everything the window does for videos and folders can be run headless.


# Quick install

YORU Tracker needs a YORU checkout **next to it**. The [Install guide]({{ site.baseurl }}/guides/01-install/) has the details.

```text
YORU-dev/
├── YORU/            # https://github.com/Kamikouchi-lab/YORU
└── YORU-Tracker/    # https://github.com/Kamikouchi-lab/YORU-Tracker
```

1. Install [uv](https://docs.astral.sh/uv/getting-started/installation/) and [Git](https://git-scm.com/).

    Windows (PowerShell):
    ```
    winget install --id=astral-sh.uv -e
    ```

2. Clone both repositories into one folder.

    ```
    mkdir YORU-dev
    cd YORU-dev
    git clone -b v2.0.0-beta.4 https://github.com/Kamikouchi-lab/YORU.git
    git clone https://github.com/Kamikouchi-lab/YORU-Tracker.git
    ```

    > YORU Tracker needs YORU 2.0.0 Beta 4 or later, the first YORU release with the external API YORU Tracker uses. `v2.0.0-beta.4` is also the YORU version YORU Tracker's CI tests against.

3. Build the environment. This installs YORU (editable, from `../YORU`) with its CUDA build of PyTorch, then YORU Tracker on top.

    ```
    cd YORU-Tracker
    uv sync
    ```

4. Launch YORU Tracker.

    ```
    uv run yoru-tracker
    ```


# Quick start

**In the window.** `uv run yoru-tracker` opens the start screen, which offers the three workflows and the tracker settings:

| Button | Use it to |
|---|---|
| **Realtime Tracking** | Run camera, detector and tracker live, with IDs, trails and latency. |
| **Video Tracking** | Track a video, step through every frame, inspect tracks, export CSV. |
| **Batch Tracking** | Track every video in a folder with one model and one set of settings. |
| **Settings** | Change the tracker settings shared by all three. |

**From the command line.**

```
uv run yoru-tracker track movie.mp4 --model best.pt --out results/            # detect + track
uv run yoru-tracker track movie.mp4 --model best.pt --out results/ --render   # + overlay video
uv run yoru-tracker track results/movie_detections.csv --config two_flies.yaml # re-track, no GPU
uv run yoru-tracker batch videos/ --model best.pt --out results/
uv run yoru-tracker config > tracker.yaml                                      # default settings
```

For `movie.mp4` you get `movie_tracks.csv` (one row per track per frame), `movie_tracks.json` (how to reproduce it) and `movie_detections.csv` (the detector's output, for re-tracking). See [Output Files]({{ site.baseurl }}/guides/06-output-files/).


# Learn about YORU Tracker

- [User Guides]({{ site.baseurl }}/guides/00-overview/) — every screen, setting and file, one page each

- [Step-by-step Protocols]({{ site.baseurl }}/tutorial/01-prepare/) — from a YORU model to an analysed tracks table

- [Tracker Settings]({{ site.baseurl }}/guides/05-tracker-settings/) — which settings to change, and when

- [Development notes]({{ site.baseurl }}/devnotes/01-tracker-api/) — the Python tracker API, plugins and testing

- [Q and A]({{ site.baseurl }}/troubleshootings/)


# Requirements

## OS
- Windows 10 or later. YORU Tracker is developed for Windows, and its CI runs on Windows. Other platforms that YORU supports are untested.

## Hardware
- The detector is YORU's, and has YORU's needs: an NVIDIA GPU with a driver that supports CUDA 12.x is recommended for detection. See YORU's [requirements]({{ site.yoru.docs }}).
- Tracking itself runs on the CPU and is light. Re-tracking stored detections needs no GPU at all.

## Software
- Python 3.10. uv installs it for you.
- [uv](https://docs.astral.sh/uv/) and [Git](https://git-scm.com/).
- A YORU checkout (≥ 2.0.0b4, < 3) beside the YORU Tracker folder.


# Status

**Implemented (v0.1.0):** application, GUI and command line; YORU detector integration; tracker API; Lite tracker with motion prediction, gating, IoU and axis cost, Hungarian assignment, recovery, lifecycle and known-population mode; video, realtime and batch tracking; CSV export with reproducibility metadata; metrics and benchmark.

**Next:** explicit occlusion state for Lite (detecting merged boxes), then the *Advanced* tracker (appearance-based re-identification when identities are ambiguous), then offline refinement (gap filling, smoothing).


# Reference

YORU Tracker builds on YORU. If you use it in your research, please cite YORU:

 - Hayato M. Yamanouchi et al., YORU: Animal behavior detection with object-based approach for real-time closed-loop feedback. Sci. Adv. 12, eadw2109 (2026). DOI: [10.1126/sciadv.adw2109](https://www.science.org/doi/10.1126/sciadv.adw2109)


# License

AGPL-3.0-or-later, as YORU. See the [LICENSE](https://github.com/Kamikouchi-lab/YORU-Tracker/blob/main/LICENSE) file for details.
