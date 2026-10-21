# Data

This folder is intentionally empty in the repository (`.gitignore`d). Place the inputs here before running the notebooks.

## CDDM benchmark (required for the full pipeline)

The evaluation uses the **CDDM** crop-disease benchmark:

> Liu, X., Liu, Z., Hu, H., Chen, Z., Wang, K., Wang, K., & Lian, S. (2024).
> *A Multimodal Benchmark Dataset and Model for Crop Disease Diagnosis.*
> ECCV 2024 Workshops. DOI: 10.1007/978-3-031-73016-0_10

Obtain the dataset from its original authors and point `DATASET_ROOT` (top of each notebook) at the image folder. The evaluation set is sampled at `MAX_PER_CLASS = 25`, yielding ~1,219 images across 53 classes and 16 crops.

## Frozen Round-1 run (optional — for analysis-only reproduction)

To reproduce the VIDA figures/tables **without** re-running inference, place a frozen Round-1 run here:

- `r1_results.csv` — per-agent, per-image predictions
- `r1_raw_bank.json` — raw Round-1 responses

Then run `notebooks/1b_vida_analysis_from_frozen_r1.ipynb` and set `R1_DATASET_DIR` to this folder.
