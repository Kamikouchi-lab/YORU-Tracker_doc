---
layout: page
title: Development and Testing
order: 2
---

Notes for working on YORU Tracker itself. The environment is the one from the [Install guide]({{ site.baseurl }}/guides/01-install/): `uv sync` creates `.venv` with YORU installed editable from `../YORU`.

---

## Source layout

```text
src/yoru_tracker/
├── app.py            command line; no arguments opens the GUI
├── core/             public API: types, TrackerBase, TrackerConfig, TrackerInfo, registry
├── tracking/         Lite (geometry, kalman, association, lifecycle, lite_tracker), baseline
├── runtime/          YORU detector adapter, sources, video, realtime, batch, stored detections
├── drawing/          overlays
├── export/           tracks CSV, reproducibility metadata
├── evaluation/       metrics, synthetic scenarios, benchmark
└── gui/              start screen and the three workflows (DearPyGui)
```

---

## Tests

```
uv run pytest                                        # the whole suite, in seconds
```

The tests that open the real window are skipped unless asked for:

```
$env:YORU_TRACKER_GUI_TESTS = "1"; uv run pytest -m gui     # PowerShell
YORU_TRACKER_GUI_TESTS=1 uv run pytest -m gui                # bash
```

Keep tests free of sleep-based timing: on Windows a sleep lasts as long as the system timer resolution allows (1 to 15.6 ms).

---

## Changing tracking behaviour

Run the benchmark before and after the change, and compare ID switches, fragmentation, IDF1 and ms/frame per scenario:

```
uv run yoru-tracker bench
```

`tests/test_evaluation.py` is a **regression gate**: it fails if any scenario gets worse than `tests/data/lite_reference.json`. After an intended improvement, regenerate the reference:

```
uv run yoru-tracker bench --json tests/data/lite_reference.json
```

Do not tune on one video. For a new failure case, add a scenario to `evaluation/scenarios.py` instead, so it is checked from then on.

---

## Rules the code keeps

- **Determinism.** Same detections + frame IDs + timestamps + configuration → equal `TrackingResult`s. No randomness, no wall-clock time inside a result.
- **Never fail silently.** No fallback from one tracker mode to another, no silent ID reset (frames out of order raise), no configuration key silently ignored (`ConfigError`).
- **One tracking path.** Realtime, video, batch and stored detections all go through `TrackerBase.update`, with the same tracker.
- **Plain results.** Public result types stay plain frozen dataclasses; internals (Kalman state, track records) never leak into them.
- **`TRACKER_API_VERSION`** changes only with an incompatible contract change.

---

## The boundary with YORU

- YORU Tracker depends on YORU; YORU never imports `yoru_tracker`.
- YORU Tracker uses only the YORU names in YORU's `docs/external_api.md`. `tests/test_yoru_boundary.py` enforces the list; if a new YORU primitive is genuinely needed, it is added to YORU's external API (and YORU's `tests/test_public_api.py`) first.
- There is no detector code in YORU Tracker: detection goes through YORU's detector registry (`yoru.libs.plugins.get_detector`, in `runtime/detection.py`). ultralytics and torch are never imported directly.
- The window is built from YORU's GUI primitives (`yoru.gui_base`, `yoru.gui_layout.GuiSession`, `yoru.libs.gui_error`), not from copies of YORU's screens.

---

## Continuous integration

CI (`.github/workflows/ci.yml`) runs on Windows. It checks YORU out beside the repository — at the repository variable `YORU_REF`, by default the `yoru-tracker-lite` branch (`YORU_REPOSITORY` names another repository; a manual run can name any YORU ref) — then runs `uv sync --locked`, the tests with the benchmark regression gate, and the benchmark table, which it posts in the job summary.

`uv sync --locked` fails when `uv.lock` no longer matches `pyproject.toml` and that YORU. **When YORU's dependencies change, run `uv lock` in YORU Tracker** and commit the new lockfile.

---

## Logs

`~/.yoru/logs/yoru_tracker.log`, beside YORU's `yoru.log` (moves with `YORU_HOME`). GUI errors, failed batch files, tracker resets and configuration loads are written there with full tracebacks — check it first when something fails.
