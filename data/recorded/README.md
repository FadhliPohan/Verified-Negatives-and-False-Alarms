# Recorded values

Values that were measured once during the original project and cannot be recomputed from the stored predictions. They are copied verbatim from the original records; nothing here was retyped by hand.

## `gpu_efficiency.csv` (Table 15)

Inference cost of the two detectors at seed 42, measured on an NVIDIA GeForce RTX 4050 Laptop GPU (6 GB) under Linux: batch 1, 640 x 640 input, bf16 autocast, 20 warm-up and 100 timed iterations; image decoding, letterboxing and preprocessing excluded. The `source` column names the original record of each row:

* YOLO11n: the efficiency benchmark run on the Ultralytics checkpoint of run `E0__seed42__7a9ae5f5` (NMS not included);
* ResNet18+FPN+FCOS: the efficiency harness of the training pipeline for run `E0b__seed42__b8f92454`, which also reports the parameter count of 12.1703 M.

The notebook reads this file in Section 14 and formats it as Table 15. Latency and memory depend on the GPU, driver and library versions; they describe the model on that machine, not a deployed system.

## `training_runs.csv`

One row per trained model, from the run record (`run.json`) the training pipeline wrote: original run id, experiment id (`E0` = YOLO11n, `E0b` = ResNet18+FPN+FCOS, `E7` = no-negatives, `E8` = size-matched), seed, configuration hash, the content hashes of the training and validation manifests the run consumed, the confidence threshold the pipeline itself selected on validation, start and end times, GPU and the PyTorch, Ultralytics or timm version.

The notebook uses the recorded training-manifest hashes to confirm that the ablation manifests it rebuilds (Table 5) are the ones the models were trained on, and the validation hashes to confirm that all runs used the same validation split. The test-manifest hash is recorded only by the YOLO11n runs; for every run the frozen test manifest is checked in Section 2 of the notebook.
