# Brain Tumour Classification from CT and MRI: Data Audit and Preprocessing

MSc project (Interim Progress Report stage). This repository contains the work completed so far: dataset auditing, leakage-safe splitting and image preprocessing. Model training, evaluation and explainability are still in progress and will be added in later commits.

## Project aim

To classify brain scans as **healthy** or **tumour** for two imaging modalities (CT and MRI), using a reproducible and leakage-aware pipeline so that reported results can be trusted.

## Dataset

- **Source:** [Brain Tumor Multimodal Image (CT & MRI)](https://www.kaggle.com/datasets/murtozalikhon/brain-tumor-multimodal-image-ct-and-mri) on Kaggle (`murtozalikhon/brain-tumor-multimodal-image-ct-and-mri`).
- **Classes:** Healthy (0) and Tumour (1).
- **Modalities:** CT and MRI.
- The notebook downloads the data automatically with `kagglehub`. The dataset is not stored in this repository.

## What this notebook covers

`Brain_Tumour_CT_MRI_preprocessing.ipynb`

1. **Setup:** installs packages, downloads the dataset, fixes random seeds (seed 42) and records software versions in `settings.json`.
2. **Data audit:** walks every image, infers modality and label from the folder names, and rejects unreadable files. For each image it records the size, mean intensity, a SHA-256 file hash, a pixel hash and a perceptual hash (pHash).
3. **Duplicate and leakage control:**
   - Exact duplicates are removed using the pixel hash.
   - Near-duplicates (pHash Hamming distance of 3 or less) are linked into similarity groups with union-find.
   - Groups with conflicting labels or modalities are quarantined and excluded.
   - An optional patient-ID mapping (`PATIENT_MAPPING`) can be supplied if the dataset provides patient IDs.
4. **Splitting:** a grouped, stratified split (`StratifiedGroupKFold`) into train, validation and test sets. All images in a group stay in the same partition. A further five-fold split is made on the development data (train and validation) for cross-validation. Assertions check that no group or pixel hash appears in more than one partition.
5. **Image preparation:**
   - Convert to greyscale, then to 3 channels.
   - Optional foreground crop, applied only if the crop keeps at least 35% of the image.
   - Resize to 224 x 224 and normalise with ImageNet mean and standard deviation.
   - Training-only augmentation: random horizontal flip and rotation of up to 10 degrees.

## Outputs

Files are written to `results/<MODE>/`:

| Folder | Contents |
|---|---|
| `audit/` | Manifests, rejected files, quarantined files, duplicates, split files |
| `tables/` | Audit summary |
| `figures/` | Class counts, split counts, preprocessing examples |
| `settings.json` | Run settings and library versions |

## How to run

```bash
pip install kagglehub numpy pandas matplotlib pillow scipy scikit-learn torch torchvision
jupyter notebook Brain_Tumour_CT_MRI_preprocessing.ipynb
```

Kaggle access may need credentials. See the [kagglehub documentation](https://github.com/Kaggle/kagglehub).

Set `MODE = 'smoke'` in the Settings cell for a quick test on a small sample (64 x 64 images). Use `MODE = 'full'` for the real run.

## Status

| Stage | Status |
|---|---|
| Data audit and cleaning | Done |
| Grouped splitting | Done |
| Preprocessing and augmentation | Done |
| Model training (CNN baselines and pretrained models) | In progress |
| Cross-validation, final test, uncertainty, Grad-CAM, robustness checks | Planned |

## Ethics and data note

The data is a public, anonymised Kaggle dataset, so no new patient data was collected. This is a research project only and is **not** a medical device or for clinical use.
