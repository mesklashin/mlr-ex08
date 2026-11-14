# Task 4 — A better leaf classifier

## Goal and evaluation

I compared changes to the Task 3 four-convolution CNN for the four leaf classes (apple, cherry, chestnut, maple). The dataset contained **136 training** and **37 validation** images, with 96×96 RGB inputs. The held-out split follows the course exercise. I trained for **15 epochs per run**, using batch size 32 and the same training/validation folders for all 10 configurations. Each configuration ran with **two random seeds (0 and 1)**. I saved the weights from the epoch with the highest standard validation accuracy, rather than simply using the final epoch.

To check robustness, I also evaluated every saved model on the **same camera-like set**, generated once from the 37 validation images with fixed random seed 42 using random crop, rotation, and brightness/contrast/saturation changes. This is a **synthetic stress test**, not a second independent validation set or a real phone photo test. Scores on these 37 transformed images are useful for comparisons but remain noisy.

## Results

The table reports the **mean over the two seeds**, with the two individual scores in parentheses. All values are percentages. The same evaluation set was used for each row.

| Variant | Validation mean (seed 0, seed 1) | Camera-like mean (seed 0, seed 1) |
|---|---:|---:|
| Scratch CNN baseline | **78.38%** (72.97, 83.78) | **58.11%** (59.46, 56.76) |
| Geometric augmentation | **70.27%** (72.97, 67.57) | **64.86%** (67.57, 62.16) |
| ColorJitter | **71.62%** (67.57, 75.68) | **52.70%** (59.46, 45.95) |
| Dropout (0.3) | **79.73%** (78.38, 81.08) | **56.76%** (56.76, 56.76) |
| AdamW | **81.08%** (72.97, 89.19) | **51.35%** (45.95, 56.76) |
| BatchNorm | **82.43%** (86.49, 78.38) | **55.41%** (56.76, 54.05) |
| Combined scratch changes | **81.08%** (86.49, 75.68) | **68.92%** (72.97, 64.86) |
| Pretrained MobileNet, frozen | **97.30%** (97.30, 97.30) | **87.84%** (83.78, 91.89) |
| Pretrained MobileNet, fine-tuned | **82.43%** (81.08, 83.78) | **79.73%** (78.38, 81.08) |
| Fine-tuned + augmentation + cosine LR | **78.38%** (75.68, 81.08) | **78.38%** (81.08, 75.68) |

All per-seed results, best epochs, and checkpoint filenames are recorded in [`task4_results.csv`](task4_results.csv). The experiment implementations are in [`task4_experiments.py`](task4_experiments.py).

## What changed and what the comparisons suggest

- **Scratch baseline:** 78.38% validation and 58.11% camera-like on average.
- **Geometric augmentation alone:** mean validation fell to 70.27%, but camera-like accuracy improved to 64.86% (+6.75 percentage points versus the scratch baseline).
- **Colour augmentation (ColorJitter) alone:** did not improve either average: 71.62% validation and 52.70% camera-like. The camera-like scores varied notably between seeds.
- **Dropout (0.3):** validation changed little (79.73% versus 78.38% baseline) and camera-like was 56.76%, slightly below baseline.
- **AdamW:** validation averaged 81.08% but ranged from 72.97% to 89.19% between seeds; the camera-like average dropped to 51.35%. The apparent validation gain is not robust.
- **Batch normalization:** validation improved to 82.43% on average, but camera-like performance was 55.41%, lower than the baseline.
- **Combined scratch changes:** validation reached 81.08%, while camera-like accuracy reached 68.92%, a +10.81 point improvement over the scratch baseline camera-like average. With only two seeds, that is encouraging but still uncertain.
- **Transfer learning, frozen MobileNetV3-Small:** achieved **97.30% validation in both seeds** and **87.84% mean camera-like**, far ahead of the scratch baseline (+18.92 validation points and +29.73 camera-like points).
- **Full fine-tuning:** yielded 82.43% validation and 79.73% camera-like; this was worse than freezing the backbone. With such a small dataset, tuning all backbone weights may be too aggressive at the chosen learning rate.
- **Fine-tuning + augmentation + cosine scheduler:** yielded 78.38% on both measures, so this combination did not beat the frozen backbone.

## Differences versus noise

Because validation includes only **37 images**, one correct prediction changes accuracy by **2.70 percentage points**; a few percentage points should not be overinterpreted. Two seeds are helpful but insufficient to estimate uncertainty precisely. The frozen MobileNet result is stronger than the scratch results by substantially more than one or two images, and 97.30% was repeated across both seeds. Its camera-like result still changed from 83.78% to 91.89% between seeds (three images), so I would not interpret the camera-like mean to hundredths of a percent as a reliable estimate of real-world accuracy.

## Which model I would put on the phone

**My preferred Task 4 model is the frozen pretrained MobileNetV3-Small**. Seed 1 reached **97.30% (36/37)** on standard validation and **91.89% (34/37)** on the fixed synthetic camera-like images; the saved checkpoint is [`task4_best_mobilenet_frozen_seed1.pth`](task4_best_mobilenet_frozen_seed1.pth). The selection of seed 1 based on the camera-like performance is exploratory because this set is derived from the same validation images and was also used for comparison.

**Actual phone testing:** I tested the original Task 2f browser-based leaf classifier on my phone, and it worked correctly; `phone_test.jpeg` documents that test. I also tested the selected Task 4 model on my phone and it worked correctly in my hands-on check. This is a qualitative phone test, not a measured accuracy score across a separate, labeled phone-photo dataset. The quantitative performance figures above come from the 37-image validation set and its fixed synthetic camera-like version.

## Reproducibility and limitations

- Experiments were performed on the group training/validation split for all variants, with two seeds, 15 epochs per run, Adam learning rate 0.001 unless AdamW is specified, and best-validation-epoch checkpoint selection.
- The camera-like inputs were generated once with seed 42; they **reuse the validation photographs** and are not genuinely new photos.
- Model selection on this validation set can make scores optimistic. A separate test set of real phone pictures would provide a more trustworthy final comparison.
- Global average pooling, label smoothing, and the optional historical dataset were not tried in this set of experiments.
- I checked the selected Task 4 model on my phone and it worked, but did not record a separate, systematically labeled phone-image accuracy metric or runtime benchmark.

## Experiment artifacts

The accompanying `task4_results.csv` records all 20 runs, `task4_experiments.py` contains the experiment code, and `task4_best_mobilenet_frozen_seed1.pth` stores the selected checkpoint. The existing Task 2f web app and model are unchanged.
