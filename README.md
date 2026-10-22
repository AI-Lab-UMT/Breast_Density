# Interpretable Breast Density Classification from Four Histogram-Derived Markers

<img width="701" height="451" alt="bd1" src="https://github.com/user-attachments/assets/aeb2ba8d-a717-47ef-9743-722a4a8915ca" />

Code and data for the article

> A. Shehu, K. Mati, R. Natowicz, *Interpretable Breast Density Classification Using Four Histogram-Derived Markers and a Shallow Neural Network* (submitted to IEEE Access).

Breast density is both a risk factor for breast cancer and a determinant of how well lesions can be seen on a mammogram: in a dense breast, a mass lies within fibroglandular tissue of similar attenuation and may be masked. The ACR BI-RADS atlas (5th edition) defines four density categories, **A** (almost entirely fatty), **B** (scattered areas of fibroglandular density), **C** (heterogeneously dense) and **D** (extremely dense).

Convolutional networks dominate automated density assessment but use hundreds of thousands to millions of parameters. This repository shows that **four markers computed from the grey-level histogram**, combined by a **neural network with fewer than 150 parameters**, capture most of the information needed, with markers that can be traced back to the BI-RADS wording.

Two tasks are addressed:

- **AB/CD**: non-dense (A or B) versus dense (C or D) breasts
- **A/B/C/D**: the four density categories

---

## Data

| Dataset | Images used | Access |
|---|---|---|
| VinDr-Mammo, Siemens Mammomat Inspiration subset | 7,623 cranio-caudal (CC) images from 3,811 women (41 A, 635 B, 5,954 C, 993 D) | [PhysioNet](https://doi.org/10.13026/br2v-7517), data use agreement required |
| VinDr-Mammo, all four devices | all CC images | same |
| INbreast (Siemens MammoNovation) | 80:20 train/test split, 61 test images | see the INbreast publication |

**The images are not redistributed here.** Only the computed marker values are provided. To recompute them, obtain VinDr-Mammo from PhysioNet under its data use agreement.

The Mammomat Inspiration subset was chosen because images from the other three devices show polarity inversions and tiled backgrounds that could not be removed without altering the breast region. The all-device experiment measures the effect of these problems.

---

## Image preprocessing

1. Linear rescaling to 0–255, correction of inverted images, mirroring of right breasts.
2. **ROI_1**: removal of the background and of background artefacts (labels, constant-value noise).
3. **ROI_2**: an automatically drawn polygon inside the breast that excludes skin folds, fingers, the contralateral breast and most of the pectoral muscle.

All markers are computed inside ROI_2.

---

## The four markers

Let *H* be the grey-level histogram of ROI_2 (*n* = 256 levels).

- One-level quantization gives a quantizer *Q₁* (the median), which splits the grey levels into a **dark** range D = [0, Q₁] and a **bright** range B = ]Q₁, 255].
- Two-level quantization (L1 error) gives quantizers *Q₂* < *Q′₂*, which define a **very dark** range VD = [0, Q₂] and a **very bright** range VB = ]Q′₂, 255].
- *S_X* denotes the Shannon entropy of the histogram restricted to range *X*, renormalised within the range; empty bins are skipped.

| Marker | Definition | Mainly separates |
|---|---|---|
| **M1** Histogram bimodality (HBM) | (Δ₁ − Δ₂) / Δ₁, the relative drop in quantization error from one to two levels | AB / CD |
| **M2** Heterogeneity of the bright range | S_B | A / BCD |
| **M3** Bright-to-dark heterogeneity ratio | S_B / S_D | AD / BC |
| **M4** Very-bright-to-bright heterogeneity ratio | S_VB / S_B | AB / CD |

> **Note on M4.** In the extraction code, `bimodality()` returns `e2` in the position of `e3`, so the column `e4/e3` equals S_VB / S_B. This is the quantity used in all experiments and described in the article.

---

## Classifier

<img width="1022" height="779" alt="model_architecture16" src="https://github.com/user-attachments/assets/caef1b7f-b6a1-4a4d-b704-a709ec636096" />

- Input: the 4 markers, standardised
- Hidden layer: 16 ReLU units
- Output: softmax with K = 2 (AB/CD) or K = 4 (A/B/C/D)
- Trainable parameters: 4·16 + 16 + 16K + K = **114** (AB/CD) and **148** (A/B/C/D)
- Training: Adam (learning rate 1e-3), focal loss (α = 0.46, γ = 2), 20 epochs, batch size 8, no early stopping, no class weighting

---

## Evaluation protocols

- **Image level**: stratified 5-fold cross-validation (`StratifiedKFold`, seed 42), 100 repetitions with the same folds and different weight initialisations (500 models per task). The scaler was fitted on the whole dataset, and the two breasts of a woman could fall in different folds.
- **Patient level**: `StratifiedGroupKFold` grouped by woman, so both CC images of a woman are always in the same fold. The scaler is fitted on the training folds only, and the folds are redrawn at each of 10 repetitions (50 models per task). The 18 women (0.5%) whose two breasts had different density categories were excluded, leaving 7,587 images from 3,793 women.

Metrics: accuracy, AUC (one-versus-rest macro average for four classes), support-weighted F1, and sensitivity. For AB/CD, C or D is the positive class. For A/B/C/D, the support-weighted sensitivity equals accuracy.

---

## Results

Values in %, mean ± standard deviation over the cross-validated models.

### Mammomat Inspiration subset (7,623 CC images)

| Task | Level | Accuracy | AUC | F1 (weighted) | Sensitivity |
|---|---|---|---|---|---|
| AB/CD | Image | 94.25 ± 0.31 | 94.82 ± 0.52 | 93.86 ± 0.37 | 98.09 ± 0.52 |
| AB/CD | Patient | 94.52 ± 0.82 | 94.89 ± 1.07 | 94.11 ± 0.95 | 98.32 ± 0.52 |
| A/B/C/D | Image | 83.18 ± 1.14 | 89.57 ± 1.76 | 81.40 ± 1.33 | 83.18 ± 1.14 |
| A/B/C/D | Patient | 83.29 ± 1.29 | 89.17 ± 2.20 | 81.38 ± 1.53 | 83.29 ± 1.29 |

All patient-level values are within one standard deviation of the image-level ones, so the image-level estimates were not inflated by the pairing of left and right breasts.

**Nested marker sets** (image level, accuracy / weighted F1):

| Markers | AB/CD | A/B/C/D |
|---|---|---|
| M1, M2 | 92.55 / 91.38 | 80.76 / 77.32 |
| M1, M2, M3 | 93.37 / 92.59 | 82.49 / 80.36 |
| M1, M2, M3, M4 | 94.25 / 93.86 | 83.18 / 81.40 |

### All four VinDr-Mammo devices (image level)

| Task | Accuracy | AUC | F1 (weighted) | Sensitivity |
|---|---|---|---|---|
| AB/CD | 91.63 ± 0.62 | 91.64 ± 0.88 | 90.68 ± 0.66 | 97.56 ± 0.85 |
| A/B/C/D | 80.20 ± 0.62 | 87.00 ± 1.26 | 78.09 ± 0.62 | 80.20 ± 0.62 |

Including images with acquisition artefacts lowered the AUC by about three points.

### INbreast (network retrained on INbreast, 61 test images)

| Task | Result |
|---|---|
| A/B/C/D | accuracy 68.9% (95% CI 56.4–79.1), quadratically weighted κ = 0.67; 15 of 19 errors between adjacent categories |
| AB/CD | accuracy 86.9% (95% CI 76.2–93.2), sensitivity 68.4%, specificity 95.2% |

The marker definitions transfer to another device, but their values shift (the average entropy is higher on INbreast), so the classifier must be retrained or recalibrated for a new acquisition system. With 148 parameters, this is inexpensive.

---

## Limitations

- At the default decision rule, the AB/CD network favours the dense majority class. On the Mammomat Inspiration subset at image level, sensitivity for C or D is 98.1% and the derived specificity for A or B is 54.8%.
- The markers were selected after inspecting the whole dataset, so cross-validation assesses the classifier but not the choice of markers.
- Categories A and B are small (41 and 635 images).
- The ablation and the all-device experiments were run at image level only. The INbreast evaluation uses a single 80:20 split.

---

## Repository contents

| File | Content |
|---|---|
| `Data_Extraction_V2.ipynb` | preprocessing (ROI_1, ROI_2) and marker extraction |
| `A_2.csv` … `D_2.csv` | marker values per density category (Mammomat Inspiration, CC); columns 0, 4, 19, 20 = M1, M2, M3, M4 |
| `markers_with_ids.csv` | same markers with `study_id`, `image_id`, `laterality` and official `split`, needed for patient-level folds |
| `Indipendence_of_Variables.ipynb` | boxplots and pairwise one-way ANOVA between categories |
| `Model for Density clasification__ (AB-CD)_(A-B-CD)_(A-B-C-D).ipynb` | image-level cross-validation |
| `train_patient_level.py` | patient-level cross-validation (grouped folds, scaler fitted on training folds) |

### Reproducing the patient-level results

```bash
pip install numpy pandas scikit-learn tensorflow
```

Run `train_patient_level.py` after building the marker table. It prints mean ± SD for each task and saves the fold-level results to `cv_patient_level_ABCD.csv` and `cv_patient_level_ABCD_binary.csv`.

---

## Citation

If you use this code, please cite the article above (citation details will be added after publication).

