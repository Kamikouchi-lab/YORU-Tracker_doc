---
layout: page
title: Install
order: 1
---

YORU Tracker is installed with [uv](https://docs.astral.sh/uv/), **next to a YORU checkout**. One `uv sync` builds an environment that holds both: YORU (installed editable from the folder beside it) and YORU Tracker on top.

---

## Prerequisites

### 1. Windows and a GPU driver

YORU Tracker is developed for **Windows 10 or later**, and its CI runs on Windows.

Detection is done by YORU, so it has YORU's needs. To detect on a GPU, install an **NVIDIA driver**: YORU v2.0.0 Beta 4 uses the CUDA 12.8 build of PyTorch, which needs **driver 570 or newer** (572.xx on Windows) and covers every card up to the RTX 50-series. The CUDA toolkit is not needed: the PyTorch wheels carry their own CUDA runtime. Check the driver with:

```
nvidia-smi
```

Without a usable GPU, detection runs on the CPU — everything works, but more slowly. Tracking itself always runs on the CPU and is light. See YORU's [Beta Install guide]({{ site.yoru.docs }}beta-guides/01-install/) for the details of drivers and compute devices.

### 2. uv and Git

Install [uv](https://docs.astral.sh/uv/getting-started/installation/) and [Git](https://git-scm.com/). In PowerShell:

```
winget install --id=astral-sh.uv -e
winget install --id=Git.Git -e
```

uv also installs Python 3.10 for you. You do not need conda or a separate Python.

---

## Install

### 1. Put YORU and YORU Tracker side by side

YORU Tracker finds YORU at `../YORU`, so both repositories must be in the **same parent folder**:

```text
YORU-dev/
├── YORU/            # https://github.com/Kamikouchi-lab/YORU
└── YORU-Tracker/    # https://github.com/Kamikouchi-lab/YORU-Tracker
```

```
mkdir YORU-dev
cd YORU-dev
git clone -b v2.0.0-beta.4 https://github.com/Kamikouchi-lab/YORU.git
git clone -b v0.1.0 https://github.com/Kamikouchi-lab/YORU-Tracker.git
```

<div class="note" markdown="1">
**Which YORU?** YORU Tracker requires **YORU 2.0.0 Beta 4 or later (and below 3)**: Beta 4 is the first YORU release with the external API that YORU documents for YORU Tracker, and the version YORU Tracker's CI tests against. The stable YORU v1.1.x is too old: with it beside YORU Tracker, `uv sync` fails.

If you already have a YORU folder, you can use it: check out the release there (`git -C YORU fetch --tags`, then `git -C YORU checkout v2.0.0-beta.4`) instead of cloning again.
</div>

### 2. Build the environment

```
cd YORU-Tracker
uv sync
```

This creates `YORU-Tracker\.venv` and installs YORU with its CUDA build of PyTorch, then YORU Tracker. The first sync downloads PyTorch and takes a while.

### 3. Check the installation

```
uv run yoru-tracker --version
```

prints `yoru-tracker 0.1.0`.

### 4. Launch

```
uv run yoru-tracker
```

`uv run python -m yoru_tracker` does the same. The start screen opens; continue with [Video Tracking]({{ site.baseurl }}/guides/02-video-tracking/).

<div class="warning" markdown="1">
**Do not run a bare `python` on Windows.** It may be the Microsoft Store stub, not the environment's Python. Use `uv run ...`, or the environment's interpreter directly: `.venv\Scripts\python.exe -m yoru_tracker`.
</div>

---

## Updating

Both folders are checked out at a release tag, so update them by checking out the newer tags, then sync again from the YORU Tracker folder. Each YORU Tracker [release](https://github.com/Kamikouchi-lab/YORU-Tracker/releases) names the YORU version it needs:

```
cd YORU-dev
git -C YORU fetch --tags
git -C YORU checkout v2.0.0-beta.4
git -C YORU-Tracker fetch --tags
git -C YORU-Tracker checkout v0.1.0
cd YORU-Tracker
uv sync
```

Replace the two tags with the ones the release you are moving to names. `git pull` does not work on a tag checkout.

---

## Where YORU Tracker keeps its files

| File | Content |
|---|---|
| `~/.yoru/logs/yoru_tracker.log` | Every error shown in the window (with its traceback), failed batch files, tracker resets, configuration loads. Next to YORU's own `yoru.log`. Rotated at 5 MB, with three old files kept. |
| `~/.yoru/yoru_tracker_gui.json` | What the window remembers between sessions: model, detector threshold, folders, camera ID, tracker settings. Delete it to start from the defaults. |

On Windows, `~` is `%USERPROFILE%`, so the log is `%USERPROFILE%\.yoru\logs\yoru_tracker.log`. Both files move with the `YORU_HOME` environment variable, as YORU's do. **Help → Open log folder** opens the folder.

Tracking results are written where you choose; see [Output Files]({{ site.baseurl }}/guides/06-output-files/).

---

## Choosing the compute device

The detector is loaded by YORU, so YORU decides where it runs: CUDA, then Apple MPS, then the CPU. To choose yourself, set `YORU_DEVICE` before launching, for example in PowerShell:

```
$env:YORU_DEVICE = "cpu"
uv run yoru-tracker
```

See [Choosing the compute device]({{ site.yoru.docs }}beta-guides/01-install/#choosing-the-compute-device) in the YORU documentation.
