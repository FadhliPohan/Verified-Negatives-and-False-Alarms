# Stored predictions

This folder is empty in the repository. The detections of the trained models are archived on Zenodo, [ISI: Zenodo DOI], as `dfire_stored_predictions.zip`. Its entries are `predictions/<run>/<split>_dfire.jsonl`, so unzipping it **inside the repository's `data/` folder** fills this folder:

```bash
cd data
unzip dfire_stored_predictions.zip      # Windows: tar -xf dfire_stored_predictions.zip
```

which gives

```
data/predictions/<run>/val_dfire.jsonl
data/predictions/<run>/test_dfire.jsonl
```

The notebook reads this layout directly. It needs the eight runs below; the archive also holds the runs of a companion study, which the notebook ignores. If the files live elsewhere, set `FIRE_PRED_DIR` to the folder that holds the run folders.

| Folder in the archive | Run id in the notebook | Detector | Seed | Training set |
|---|---|---|---|---|
| `yolo11n_seed42` | `yolo11n_seed42` | YOLO11n | 42 | full |
| `yolo11n_seed1337` | `yolo11n_seed1337` | YOLO11n | 1337 | full |
| `yolo11n_seed2024` | `yolo11n_seed2024` | YOLO11n | 2024 | full |
| `resnet18fpn_seed42` | `resnet18fpn_seed42` | ResNet18+FPN+FCOS | 42 | full |
| `resnet18fpn_seed1337` | `resnet18fpn_seed1337` | ResNet18+FPN+FCOS | 1337 | full |
| `resnet18fpn_seed2024` | `resnet18fpn_seed2024` | ResNet18+FPN+FCOS | 2024 | full |
| `tanpa_negatif_seed42` | `no_negatives_seed42` | ResNet18+FPN+FCOS | 42 | positives only (7,938 images) |
| `kontrol_ukuran_seed42` | `size_matched_seed42` | ResNet18+FPN+FCOS | 42 | 4,243 positives + 3,695 negatives |

Folders named after the run ids of the notebook are accepted as well.

## File format

JSON lines, UTF-8. The first line may be a metadata record; every other line describes one image of the split, and every image of the manifest appears exactly once (the notebook refuses incomplete files).

```json
{"_meta": {"split": "test", "domain": "dfire", "experiment_id": "E0", "n_samples": 3257, "conf_threshold_used": 0.05, "inference_seconds": 29.07}}
{"sample_id": "dfire/test/images/AoF06733", "source": "dfire", "detections": [{"bbox": [348.87, 177.18, 379.38, 264.67], "score": 0.05311, "class_idx": 1, "class": "smoke"}]}
{"sample_id": "dfire/test/images/AoF06734", "source": "dfire", "detections": []}
```

* `sample_id` matches the `sample_id` column of the manifests.
* `bbox` is `[x1, y1, x2, y2]` in pixels of the curated image, whose size is the `width` and `height` of the manifest row (the curated copy capped the long side at 1,280 pixels).
* `score` is the detector confidence rounded to five decimals; only detections with a score of at least 0.05 were stored, after class-aware NMS at IoU 0.6.
* `class` is `fire` or `smoke` (`class_idx` 0 or 1).

## Checksums

SHA-256 of each file as it appears under `data/predictions/` after unzipping.

| File | Bytes | SHA-256 |
|---|---:|---|
| `yolo11n_seed42/val_dfire.jsonl` | 1,059,005 | `6d9972f11e236f95a800192472c8b594790610896dc4c031d27ab1fa787703da` |
| `yolo11n_seed42/test_dfire.jsonl` | 1,055,955 | `fbccc2f969d5831c1b41198aa6291a77b35ab810249845cffc5677bd40b41f34` |
| `yolo11n_seed1337/val_dfire.jsonl` | 1,102,960 | `81603fe040fcb1fb55e0ead9d5721d01af4432edfafaac122bf1f6d4971e92a9` |
| `yolo11n_seed1337/test_dfire.jsonl` | 1,107,052 | `a848729573f9ed0c82295a8324ce26c5cb44036b3d8e593537b7adca9ecb5485` |
| `yolo11n_seed2024/val_dfire.jsonl` | 1,081,088 | `45c1ffe32b9e9c8d8a7a2092d0c9176a6502b9c29da254f70419b2f6f5a0a11c` |
| `yolo11n_seed2024/test_dfire.jsonl` | 1,086,594 | `c8a0c4653b79e03b9bd27d4f1f05b5ffa58f0845124c9c672590ed366d40f6da` |
| `resnet18fpn_seed42/val_dfire.jsonl` | 71,274,167 | `975f8caf0a3846ede15fce9fc024fb483c570ff44754f70875d5112eb88f225f` |
| `resnet18fpn_seed42/test_dfire.jsonl` | 67,133,472 | `983ba6483fd9b93c669fe41b6ee4f615968d498801a92253f7d3937ee8005b21` |
| `resnet18fpn_seed1337/val_dfire.jsonl` | 48,204,836 | `fff0055af2bfe03149446d3ce4fbbecf37fa0511e4768e0bcefb6b05a7b3de46` |
| `resnet18fpn_seed1337/test_dfire.jsonl` | 44,960,344 | `fb8f90e4fe595ffee781dcecd483b0e5754a5e1c4449f93d0176976bd458daec` |
| `resnet18fpn_seed2024/val_dfire.jsonl` | 50,604,604 | `f9e0a3d07bafc167fbe103aec847db4f4f96e85795cd33922143cd9a7c3c4e7c` |
| `resnet18fpn_seed2024/test_dfire.jsonl` | 47,389,700 | `88e09fd62381e5bfa66a5455e7392dead6d7c8b9563b34008dccc3f5d98dfb22` |
| `tanpa_negatif_seed42/val_dfire.jsonl` | 83,651,104 | `8c77804e091ea9a94b0df56c528d475d63113c6116d74392eb3660da0cb3027c` |
| `tanpa_negatif_seed42/test_dfire.jsonl` | 78,652,478 | `b04281d073c4c2ca27d5a90e179642b36c14f61b45076fbce8f006ec4a59d9e9` |
| `kontrol_ukuran_seed42/val_dfire.jsonl` | 83,571,314 | `96047121311fe088ff00f5e21ce1f866c823aaf9e4eab65fa3f11e525b2d2863` |
| `kontrol_ukuran_seed42/test_dfire.jsonl` | 78,667,210 | `1c7339eec94814af3f2a57f47d5ae62a4cd3e8e1d04ada5566050ae04d144919` |
