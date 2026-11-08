# Dominant frequency of the preoperative surface ECG for predicting AF termination after the Cox-Maze IV procedure

Analysis code for the manuscript:

> Dominant frequency of the preoperative surface electrocardiogram for predicting termination of atrial fibrillation after the Cox-Maze IV procedure (submitted).

The code computes the dominant frequency (DF) of atrial fibrillation from lead II of preoperative 12-lead electrocardiograms (ECGs) after QRST cancellation. It then evaluates prediction models for AF termination at discharge using repeated patient-grouped cross-validation and patient-level cluster bootstrap.

## Contents

| File | Description |
|---|---|
| `01_ECG-FM_extraction.ipynb` | Extracts ECG foundation model (ECG-FM) embeddings from the 12-lead ECGs |
| `02_DF_analysis.ipynb` | DF calculation, quality assessment of QRST cancellation, prediction models, sensitivity and exploratory analyses, and figures |

Order of execution: (1) `01_ECG-FM_extraction.ipynb` → (2) `02_DF_analysis.ipynb`

The main analysis is in `02_DF_analysis.ipynb`. `01_ECG-FM_extraction.ipynb` is needed only for the models including ECG-FM embeddings; without it, these models are skipped and all other analyses run.

## Data availability

The ECG recordings and clinical data are **not included** in this repository. They contain patient information, and the ethics approval for this study does not cover public release. The notebooks are provided so that the methods can be inspected and applied to other datasets in the input format described below.

## Expected input

Set `ROOT` at the top of both notebooks to your project folder. The notebooks expect the following layout:

```
ROOT/
├── data/
│   ├── metadata.csv      # one row per ECG (mapping table)
│   ├── table.xlsx        # patient table (Section 9-4 only)
│   └── ECG/              # one CSV file per ECG
├── ckpts/                # ECG-FM checkpoint (downloaded automatically)
└── ecgfm/                # embeddings written by 01_ECG-FM_extraction.ipynb
```

**`metadata.csv`**: one row per ECG, with the following columns:

| Column | Description |
|---|---|
| `ecg_path` | Path to the ECG CSV file |
| `ID` | Patient identifier (several ECGs per patient are allowed) |
| `Result` | Outcome: 1 = AF termination (sinus rhythm at discharge), 0 = non-termination |
| `Age` | Age at surgery (years) |
| `LAD` | Left atrial diameter (mm) |
| `Duration` | Days from the ECG to surgery |

**ECG CSV files**:
- One column per lead, named `Ⅰ, Ⅱ, Ⅲ, aVR, aVL, aVF, V1–V6`. In our data, leads I–III use full-width Roman numerals; edit `LEAD`, `LEADS_12` and `ECG_FM_LEAD_ORDER` to match your files.
- Sampled at 500 Hz.
- Amplitude in A/D counts (4.88 μV/count; see `UV_PER_COUNT`).
- Encoded in UTF-8 (with or without BOM).

**`table.xlsx`**: patient table used only in Section 9-4 (DF by AF type). The column positions in that cell are specific to our table and must be adapted.

## Environment

- Analysis notebook: Python 3.10, NumPy, pandas, SciPy, scikit-learn, matplotlib, tqdm, openpyxl
- ECG-FM extraction: PyTorch, fairseq-signals, ecg-transform, huggingface_hub. Install them following the instructions in the ECG-FM repository (https://github.com/bowang-lab/ECG-FM).

Both notebooks were run in a single conda environment and tested with Python 3.10.20, NumPy 2.2.5, pandas 2.3.3, SciPy 1.15.3, scikit-learn 1.7.2, PyTorch 2.12.0 and fairseq-signals 1.0.0a0+f8f0ff1. The package versions are also printed by the last section of each notebook.

## Main settings (prespecified)

- DF: lead II; 0.5–40 Hz band-pass filter; multi-template QRST cancellation; Hanning window; FFT length 65,536; search band 3.5–9.0 Hz
- Models: L2-regularized logistic regression (C = 1.0); stratified group 5-fold cross-validation repeated 10 times (patients never split across folds); out-of-fold predictions averaged across repetitions
- Inference: patient-level cluster bootstrap (2,000 resamples)
- ECG-FM: pretrained checkpoint `mimic_iv_ecg_physionet_pretrained.pt` (not fine-tuned); 768-dimensional Transformer embedding reduced to 3 principal components, fitted within each training fold

## License

MIT License (see `LICENSE`).

The ECG-FM model and its checkpoint are distributed by their original authors under their own license and are not included here.
