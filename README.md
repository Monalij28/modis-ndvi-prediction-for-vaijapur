# MODIS NDVI forecasting for Vaijapur tehsil with ConvLSTM

Code and data for: *Jadhav, Deshmukh, Gaikwad & Vibhute — ConvLSTM next-composite forecasting of MODIS MOD13Q1 NDVI for Vaijapur tehsil, Maharashtra* (under review).

## Contents

| File | Purpose |
|---|---|
| `gee_export_mod13q1_vaijapur.js` | Google Earth Engine script: MOD13Q1 v6.1 NDVI, SummaryQA masking (0–1 kept), scale factor 0.0001, export of `ndvi_stack.tif` and `NDVI_Timestamps.csv` (1 Jan 2014 – 18 Dec 2024, 253 composites) |
| `ndvi_stack.tif` | NDVI stack used in the paper: 253 composites × 19 × 23 pixels (250 m, MODIS sinusoidal), **without** SummaryQA filtering |
| `NDVI_Timestamps.csv` | Composite date of every band |
| `ConvLSTM_NDVI_Vaijapur_fixed.ipynb` | Full experiment: causal gap filling, training-only scaling, chronological 70/10/20 train/validation/test split, ConvLSTM and CNN (5 seeds), persistence / seasonal-naïve / climatology baselines, block-bootstrap CIs, figures and tables |
| `results/` | Output of the run reported in the paper (tables, figures, `run_info.json`, `results_summary.md`) |
| `requirements.txt` | Python packages |

## Reproduce

1. To reproduce the paper, use `ndvi_stack.tif` and `NDVI_Timestamps.csv` from this repository. *(Optional)* `gee_export_mod13q1_vaijapur.js` exports a QA-filtered stack (SummaryQA ≤ 1) for follow-up work; results with that stack will differ from the paper.
2. Open `ConvLSTM_NDVI_Vaijapur_fixed.ipynb` in Google Colab, choose a GPU runtime, upload `ndvi_stack.tif` and `NDVI_Timestamps.csv`, and run all cells.
3. All reported numbers are written to `results/results_summary.md`; versions and settings to `results/run_info.json`.

Neural-network scores vary slightly between hardware and library versions; the paper reports mean ± SD over five seeds for this reason.

## Protocol summary

- Input: 5 consecutive composites → target: next composite (~16 days ahead).
- Windows: N = 253 − 5 = 248. Training 173 (targets 22 Mar 2014 – 14 Sep 2021), validation 25 (30 Sep 2021 – 16 Oct 2022), test 50 (1 Nov 2022 – 18 Dec 2024). The test set is used once.
- Min–max scaling and climatology use training-period composites only. Gaps are filled with each pixel's last valid past value. No spatial smoothing.
- Scores in NDVI units on non-missing target pixels; SSIM averaged over all test targets.

## Main result (test set, 50 composites)

ConvLSTM RMSE 0.103 ± 0.004 (5 seeds) vs persistence 0.116, seasonal naïve 0.110, CNN 0.116 ± 0.025 and per-pixel climatology 0.095. The ConvLSTM beats persistence (skill 0.21, 95 % CI 0.08–0.38) but not climatology.

## Data source

MODIS/Terra Vegetation Indices 16-Day L3 Global 250 m (MOD13Q1 v6.1), Didan, K. (2021), NASA EOSDIS LP DAAC, accessed through the Google Earth Engine catalogue (`MODIS/061/MOD13Q1`).
