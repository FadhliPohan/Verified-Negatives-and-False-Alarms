# Verified Negatives and False Alarms in Fire and Smoke Detection: A Cost-Sensitive Re-Evaluation on the D-Fire Benchmark

**Muhammad Fadhli Dzil Ikram** ([ORCID 0009-0008-1748-8715](https://orcid.org/0009-0008-1748-8715), corresponding author, 09012682529009@student.unsri.ac.id) and **Samsuryadi**
Master of Computer Science, Faculty of Computer Science, Sriwijaya University, Palembang, Indonesia

Repository: <https://github.com/FadhliPohan/Verified-Negatives-and-False-Alarms>

Code and data for the paper of the same title, submitted to *Pattern Analysis and Applications* (Springer).

The paper asks what a single mAP50 score leaves out when a fire and smoke detector is used as an alarm. D-Fire contains 9,838 images that a person has verified to contain neither fire nor smoke, so the alarm rate on fire-free scenes can be measured rather than guessed. On a frozen test split, with three seeds per model, YOLO11n has the higher mAP50 (0.7114 against 0.6646 for a ResNet18+FPN+FCOS detector) but raises 3.5 times as many false alarms, and under an expected-cost model the ranking of the two detectors reverses at every miss-to-false-alarm cost ratio from 1 to 50. A three-condition ablation shows that removing the verified negatives from training leaves mAP50 within seed noise while multiplying the alarm rate by 48.8.

This repository reproduces every computed table and figure of the paper from the stored detections of the eight trained models. One notebook does all of it, on a CPU, in a few minutes.

## What is in the repository

| Path | Content |
|---|---|
| `verified_negatives.ipynb` | The analysis: data checks, metrics, Tables 2-15, Figures 1-8, an automated comparison with the manuscript, and an optional appendix with the training code of both detectors |
| `data/manifests/` | Frozen train / validation / test manifests of the curated D-Fire split (Parquet) with their content hashes (`MANIFEST_HASHES.json`) and the split report (`BUILD_SUMMARY.json`) |
| `data/curation/` | Recipe and report of the curation step that produced the curated copy of D-Fire (provenance only; not read by the notebook) |
| `data/recorded/` | Values measured on the GPU that cannot be recomputed on a CPU (Table 15) and the records of the eight training runs (see `data/recorded/README.md`) |
| `data/predictions/` | Empty; the stored detections go here after download (see below) |
| `requirements.txt` | Packages needed to run the analysis |
| `requirements-training.txt` | Additional packages for the optional training appendix |
| `CITATION.cff`, `LICENSE` | How to cite this repository; MIT licence of the code |

## Installation

Python 3.10 or newer. The notebook was validated with Python 3.11, NumPy 2.4, pandas 3.0, PyArrow 24, Matplotlib 3.11 and Pillow 12.

```bash
git clone https://github.com/FadhliPohan/Verified-Negatives-and-False-Alarms.git
cd Verified-Negatives-and-False-Alarms
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

The training appendix additionally needs PyTorch, torchvision, timm, Ultralytics and OpenCV (`pip install -r requirements-training.txt`). It is switched off by default and none of the tables depend on it.

## Getting the data

**Manifests.** Included in `data/manifests/`. The notebook verifies that the test manifest is the frozen one (content hash `6246e1bcca79...`) before computing anything.

**Stored predictions (required).** The detections that the trained models wrote on the validation and test splits are archived on Zenodo, [ISI: Zenodo DOI], as a single file, `dfire_stored_predictions.zip` (about 660 MB of JSON lines for the runs of this paper). The archive's entries are `predictions/<run>/<split>_dfire.jsonl`, one folder per run under the run's stored folder name. Download it and unzip it **inside the repository's `data/` folder**:

```bash
cd data
unzip dfire_stored_predictions.zip      # Windows: tar -xf dfire_stored_predictions.zip
cd ..
```

This creates `data/predictions/<run>/<split>_dfire.jsonl`. The notebook needs these eight run folders:

```
data/predictions/
    yolo11n_seed42/          val_dfire.jsonl  test_dfire.jsonl
    yolo11n_seed1337/        val_dfire.jsonl  test_dfire.jsonl
    yolo11n_seed2024/        val_dfire.jsonl  test_dfire.jsonl
    resnet18fpn_seed42/      val_dfire.jsonl  test_dfire.jsonl
    resnet18fpn_seed1337/    val_dfire.jsonl  test_dfire.jsonl
    resnet18fpn_seed2024/    val_dfire.jsonl  test_dfire.jsonl
    tanpa_negatif_seed42/    val_dfire.jsonl  test_dfire.jsonl    (no-negatives ablation)
    kontrol_ukuran_seed42/   val_dfire.jsonl  test_dfire.jsonl    (size-matched ablation)
```

The archive also holds the runs of a companion study; the notebook ignores them. Inside the notebook and in its output tables the two ablation runs are called `no_negatives_seed42` and `size_matched_seed42`; folders under those names are accepted too. If the predictions live elsewhere, set `FIRE_PRED_DIR` to the folder that contains the run folders. File format, sizes and SHA-256 checksums are listed in `data/predictions/README.md`.

**D-Fire images (optional).** Only Figure 7 shows images. Download the dataset from <https://github.com/gaiasd/DFireDataset> and set `FIRE_DFIRE_IMAGES` to the folder that contains `train/images` and `test/images`. Without it, Figure 7 is skipped with a message, and the rule-based choice of its three example images is still computed and checked.

## Running the analysis

Interactively:

```bash
jupyter lab verified_negatives.ipynb
```

and run all cells. Non-interactively, executing every cell and saving the outputs into a copy of the notebook:

```bash
jupyter nbconvert --to notebook --execute verified_negatives.ipynb --output verified_negatives_executed.ipynb
```

Paths are taken from environment variables, with defaults relative to the notebook:

| Variable | Default | Meaning |
|---|---|---|
| `FIRE_DATA_DIR` | `data` | folder with `manifests/`, `recorded/` and, by default, `predictions/` |
| `FIRE_PRED_DIR` | `$FIRE_DATA_DIR/predictions` | stored detections, one folder per run |
| `FIRE_DFIRE_IMAGES` | unset | D-Fire root, only for Figure 7 |
| `FIRE_OUTPUT_DIR` | `outputs` | where tables (`outputs/tables/*.csv`) and figures are written; every figure is saved as `FigN.png` and as a vector `FigN.pdf` |

**Expected runtime:** about 2 to 4 minutes on a laptop CPU (2.2 minutes measured), with peak memory below 1 GB. Most of the time goes to parsing the prediction files and to the two bootstrap procedures (2,000 resamples each, seed 42, so results are identical from run to run).

The last section compares 2,306 computed values with the numbers printed in the manuscript, with the full-precision outputs of the original analysis and, for Figure 7, with the figure as drawn. It prints a PASS/FAIL summary per table and figure and stops with an error if anything differs. It also checks 14 qualitative statements of the manuscript against the data and lists any that the data do not support.

## Where each result of the paper comes from

| Paper item | Content | Notebook section | Output file |
|---|---|---|---|
| Table 1 | Literature survey | not computed | - |
| Table 2 | Dataset composition per split | 4 | `table02_dataset_composition.csv` |
| Table 3 | Object scale by class | 4 | `table03_object_scale.csv` |
| Table 4 | Training configuration | static; used by the appendix | - |
| Table 5 | Ablation training conditions (manifests rebuilt and hash-checked) | 4 | `table05_ablation_conditions.csv` |
| Table 6 | Test results at the validation-selected threshold | 5 | `table06_main_results.csv` |
| Table 7 | Per-seed mAP50, false-alarm rate, threshold | 5 | `table07_per_seed.csv` |
| Table 8 | False alarms at fixed recall, paired intervals | 7, 9 | `table08_far_at_fixed_recall.csv` |
| Table 9 | Precision against the alarm rate | 5 | `table09_precision_vs_far.csv` |
| Table 10 | Expected cost, paired intervals | 8, 9 | `table10_expected_cost.csv` |
| Table 11 | Bootstrap intervals for the alarm rate | 9 | `table11_far_bootstrap.csv` |
| Table 12 | Size of missed versus caught objects | 10 | `table12_missed_vs_caught.csv` |
| Table 13 | Negative-image ablation | 13 | `table13_negative_ablation.csv` |
| Table 14 | Double dissociation | 13 | `table14_double_dissociation.csv` |
| Table 15 | Computational cost on the GPU | 14 (recorded) | `table15_compute_cost.csv` |
| Figure 1 | Fire versus smoke AP; object scale | 6 | `Fig1.png`, `Fig1.pdf` |
| Figure 2 | False alarms against recall | 7 | `Fig2.png`, `Fig2.pdf` |
| Figure 3 | Expected cost against the cost ratio | 8 | `Fig3.png`, `Fig3.pdf` |
| Figure 4 | Per-seed mAP50 and false-alarm rate | 5 | `Fig4.png`, `Fig4.pdf` |
| Figure 5 | Median size of missed and caught objects | 10 | `Fig5.png`, `Fig5.pdf` |
| Figure 6 | Confidence calibration | 11 | `Fig6.png`, `Fig6.pdf` |
| Figure 7 | Qualitative examples (needs the images); annotations dashed, detections solid | 12 | `Fig7.png`, `Fig7.pdf` |
| Figure 8 | Ablation: mAP50 and false-alarm rate | 13 | `Fig8.png`, `Fig8.pdf` |
| Equations 1, 2 | False-alarm rate; expected cost | 3, 8 | - |
| Section III-H | Reproduction check of the analysis code | 3, 15 | `run_metrics.csv` |

Further intermediate tables (per-run metrics, threshold selection, full false-alarm curves, all three cost policies, error listing, calibration bins, the comparison log) are written next to these.

## What cannot be recomputed here

* **Table 15** (latency, frames per second, peak GPU memory) was measured on an NVIDIA GeForce RTX 4050 Laptop GPU with batch 1, 640 x 640 input and bf16 autocast. Those numbers depend on the hardware, so the notebook reads them from `data/recorded/gpu_efficiency.csv` instead of recomputing them. The parameter count of the in-house detector is checked independently by the smoke test of the training appendix.
* **The trained weights and the training runs themselves.** The analysis starts from the stored detections. The appendix contains the code to retrain both detectors and to export new predictions in the same format: Ultralytics YOLO11n with the locked hyperparameters (batch 8 with Ultralytics' default nominal batch size `nbs` = 64, i.e. 8-step gradient accumulation and an effective batch of 64), and a port of the ResNet18+FPN+FCOS detector with its loss, target assignment and post-processing (batch 8 with 2-step gradient accumulation, an effective batch of 16). Retraining takes several GPU hours per run and will not reproduce the stored predictions bit for bit; it should reproduce the conclusions within the seed spread of Table 6. Only the CPU smoke test of the appendix has been run for this release.
* **Table 1** is a reading of nine published papers, not a computation.

## How to cite

The paper:

> Dzil Ikram, M. F., & Samsuryadi (2026). Verified Negatives and False Alarms in Fire and Smoke Detection: A Cost-Sensitive Re-Evaluation on the D-Fire Benchmark. Manuscript submitted to *Pattern Analysis and Applications*. [ISI: DOI of the paper once published]

```bibtex
@unpublished{dzilikram2026verified,
  author = {Dzil Ikram, Muhammad Fadhli and Samsuryadi},
  title  = {Verified Negatives and False Alarms in Fire and Smoke Detection:
            A Cost-Sensitive Re-Evaluation on the {D-Fire} Benchmark},
  note   = {Manuscript submitted to Pattern Analysis and Applications. [ISI: DOI of the paper once published]},
  year   = {2026}
}
```

The code (this repository; machine-readable metadata in `CITATION.cff`):

> Dzil Ikram, M. F., & Samsuryadi (2026). *Verified Negatives and False Alarms in Fire and Smoke Detection: code and analysis notebook* [Computer software]. https://github.com/FadhliPohan/Verified-Negatives-and-False-Alarms

The stored predictions:

> Dzil Ikram, M. F., & Samsuryadi (2026). *Stored detections of the D-Fire fire and smoke detectors* (`dfire_stored_predictions.zip`) [Data set]. Zenodo. [ISI: Zenodo DOI]

Please also cite the D-Fire dataset as its authors ask (see <https://github.com/gaiasd/DFireDataset>).

## Licence

* **Code** (the notebook and everything else in this repository): MIT, see `LICENSE`.
* **Stored predictions** on Zenodo: Creative Commons Attribution 4.0 International (CC BY 4.0).
* **D-Fire images and annotations** keep their own licence; see <https://github.com/gaiasd/DFireDataset>. The manifests in `data/manifests/` list the D-Fire images and their boxes and are provided only to define the split used in the paper.
* Ultralytics, used only by the optional training appendix, is licensed under AGPL-3.0.
