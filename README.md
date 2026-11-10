# CalLIOPE

**Cal**cium **L**ive-imaging **O**utput **P**ipeline for **E**pileptiform-recordings.

A desktop GUI that takes raw two-photon calcium-imaging TIFFs and turns
them into cells, dF/F traces, events, clusters, and propagation maps.
Every step is a tab; you click through them left to right. It was built
for epileptiform activity in brain slices, but the stages are general
calcium-imaging analysis and every parameter is editable.

## Requirements

- **Python 3.11 or 3.12** (not 3.10 or earlier, not 3.13+).
- **OS:** developed and tested on Windows 10/11. Linux installs the same
  way (plus Tk, see below); macOS is untested.
- **GPU (optional):** an NVIDIA GPU with CUDA speeds up Suite2p, Cellpose
  and the cell filter (via PyTorch), and Tab 3's dF/F and Tab 7's
  cross-correlation (via CuPy). Everything also runs on the CPU, just more
  slowly.
- **Disk:** a few GB for the install, plus room for your recordings.
  Detection writes large intermediate files (tens of GB for a long
  recording) and prunes most of them when it finishes. Keep the install
  and your data on a local drive, not a network drive.

## Install

**1. Get the code.** Either click the green **Code** button on GitHub →
**Download ZIP** and unzip it, or:

```bash
git clone https://github.com/Richard0816/calliope.git
```

**2. Install.** Open a terminal in the unzipped/cloned folder and run:

```bash
python -m venv venv
venv\Scripts\activate              # Windows
# source venv/bin/activate         # macOS / Linux
pip install -r requirements.txt
pip install -e . --no-deps
```

The first line must use Python 3.11 or 3.12. On Windows,
`py -3.12 -m venv venv` picks it explicitly. Check with
`python --version` after activating.

`requirements.txt` installs the CPU build of PyTorch on Windows and
macOS. For NVIDIA GPU acceleration, the optional NWB export, a `uv` or
conda install, see [docs/install.md](docs/install.md).

**Linux:** install Tk first (`sudo apt install python3.12-tk`, matching
your Python), since pip can't. The steps above pull a ~5.5 GB CUDA build
of PyTorch on Linux; for a small CPU-only install see
[docs/install.md](docs/install.md).

## Quick start

1. **Launch** with the `calliope` command (from the activated venv).
2. **Tab 1 — Input & Preprocess:** pick the folder holding your TIFFs,
   select one or more TIFFs (several selected = one recording), pick an
   output root, and click **Run preprocessing**.
3. **Tab 2 — QC Preview:** check the movie and mean image for drift or
   motion before the heavy compute.
4. **Tab 3 — Suite2p Detection:** check the frame rate, µm/px and GCaMP
   variant, then click **Run detection + cell filter**. The first run
   downloads the Cellpose model (~1.2 GB, one time).
5. Work through **Tabs 4–8** in order. Each tab picks up the previous
   tab's results automatically.

To reopen a finished recording later, use **Open run folder…** in the
sidebar; **Import Suite2p output…** builds a CalLIOPE run from an
existing Suite2p result. To process many recordings unattended, use
**Tab 0 — Batch runner**.

## The tabs

0. **Batch runner** — queues recordings and runs Tabs 1 and 3–8 on each
   one unattended, with per-recording parameters, figure export and a CSV
   report.
1. **Input & Preprocess** — intensity shift to a non-negative uint16 TIFF, mean image, QC GIF.
2. **QC Preview** — inspect the QC movie and mean image before heavy compute.
3. **Suite2p Detection** — Suite2p registration + Suite2p/Cellpose ROI extraction, dF/F, cell-filter model, ROI curation, NWB export.
4. **Low-pass filter** — Butterworth low-pass + Savitzky-Golay derivative.
5. **Event detection** — per-ROI onsets and population event windows.
6. **Clustering** — hierarchical clustering of filtered traces.
7. **Cross-correlation** — cluster × cluster lead/lag analysis with a circular-shift null, optional GPU via CuPy.
8. **Spatial propagation** — per-event activation-order maps and a directional propagation test.

## Where outputs go

Each recording gets its own folder under the output root you pick in
Tab 1 (by default a `data` folder next to your TIFF folder):

```
<output root>/<recording>/
├── shifted_<name>.tif, mean.npy, qc.gif   Tab 1
├── detection/final/suite2p/plane0/        Tabs 3-7: Suite2p arrays (F, Fneu, spks,
│                                          stat, iscell, ops), dF/F and low-pass
│                                          memmaps, meta.json, calliope_summary.xlsx,
│                                          cluster and cross-correlation results
└── calliope_figures/                      exported figures + manifest.json
```

A batch run also writes `calliope_batch.json` (the queue) and
`batch_report.csv` to its output folder. The per-tab READMEs list every
file.

## Documentation

- [Pipeline walkthrough](src/calliope/pipeline_gui_walkthrough.md) — what
  each tab does and why, for someone new to calcium imaging.
- [Install options](docs/install.md) — uv, conda, GPU acceleration,
  NWB export.
- [Troubleshooting](docs/troubleshooting.md) — install errors, Detection
  failures, slow first runs.
- Each tab folder under `src/calliope/tabs/` has a `README.md` covering
  the maths and parameters.

## Citation

Citation: STAR Protocols manuscript in preparation. Until it is
published, please cite this repository
(https://github.com/Richard0816/calliope).

CalLIOPE builds on [Suite2p](https://github.com/MouseLand/suite2p) and
[Cellpose](https://github.com/MouseLand/cellpose); please cite them too.

## License

MIT — see [LICENSE](LICENSE). Copyright (c) 2026 Richard Jiang.
