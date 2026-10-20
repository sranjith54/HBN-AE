# HBN-AE: Hybrid Bayesian Network – Autoencoder Intrusion Detection System

HBN-AE is a two-stage, interpretable intrusion detection system for software-defined and enterprise networks. In the first stage, a K-dependence Bayesian Network (KDB-BN) learned over discretised flow features produces calibrated class posteriors whose dependency structure can be inspected directly. In the second stage, an autoencoder trained on benign traffic contributes a reconstruction-error signal that captures deviations from normal behaviour. The standardised reconstruction error acts as an anomaly gate (normal vs. attack), the Bayesian posterior may confirm or override that gate, and the attack type is assigned by the KDB-BN, so the system combines the transparency of probabilistic graphical models with the representational capacity of deep feature learning. The repository contains the full experimental pipeline, five baseline models (SAE+SMR, AE+SVM, AE+Gaussian+NB, MLP, RNN) and a KDB-BN-only ablation, evaluated over five random seeds on two public datasets.

## Datasets

| Dataset | Description | Download |
|---|---|---|
| **UNR-IDD** | SDN intrusion detection dataset built from port statistics; 6 classes (Normal, PortScan, TCP-SYN, Blackhole, Diversion, Overflow) | https://www.tapadhirdas.com/das-lab/datasets/unr-idd |
| **CSE-CIC-IDS2018** | Enterprise flow dataset (CICFlowMeter features); four capture days (14, 15, 16 and 21 February 2018) are used | https://www.unb.ca/cic/datasets/ids-2018.html |

The datasets are not redistributed in this repository (size and licensing). Place them as described in *How to run*.

## Key results (HBN-AE, 5 seeds, mean ± std)

| Dataset | Accuracy (%) | Macro-F1 (%) | Macro AUC (OvR, %) |
|---|---|---|---|
| UNR-IDD | 91.81 ± 0.63 | 91.90 ± 0.81 | 99.40 ± 0.07 |
| CIC-IDS2018 | 99.76 ± 0.05 | 99.78 ± 0.05 | 99.99 ± 0.01 |

Full per-model tables (including all baselines, weighted precision/recall/F1, training time and inference latency) are in `results/unridd/results.md` and `results/cic2018/results.md`.

Reference environment: Python 3.11.6, TensorFlow 2.21.0, scikit-learn 1.9.1, Intel Xeon 6975P-C (32 cores, CPU only), 247 GB RAM.

## Repository structure

```
HBN_AE_GitHub/
├── src/
│   ├── run_unridd.py          # Full experiment on UNR-IDD (HBN-AE + baselines + ablation)
│   ├── aggregate_unridd.py    # Aggregates per-seed results -> summary.json, results.md, figures
│   ├── prep_cic2018.py        # CIC-IDS2018 cleaning, de-duplication and stratified subsetting
│   ├── run_cic2018.py         # Full experiment on CIC-IDS2018
│   └── aggregate_cic2018.py   # Aggregation for CIC-IDS2018
├── results/
│   ├── unridd/                # results_s{0..4}.json, preds_seed{0..4}.npz, summary.json, results.md
│   └── cic2018/               # same layout for CIC-IDS2018
├── figures/
│   ├── unridd/                # ROC, PR, confusion matrix, metric bars, latency, BN structure, ...
│   └── cic2018/
├── models/                    # Reserved for serialised models (models are trained in memory; see note)
├── data/                      # Place datasets here (git-ignored)
│   ├── unridd/
│   └── cic2018/
├── requirements.txt
├── LICENSE
└── README.md
```

## How to run

### Step 1 – Install requirements

```bash
pip install -r requirements.txt
```

### Step 2a – UNR-IDD

1. Download the dataset and save it as `data/unridd/UNR-IDD.csv`.
2. Run the experiment and aggregate the results:

```bash
python src/run_unridd.py
python src/aggregate_unridd.py
```

### Step 2b – CIC-IDS2018

1. Download the four day CSVs (Wednesday 14-02-2018, Thursday 15-02-2018, Friday 16-02-2018 and Wednesday 21-02-2018, `*_TrafficForML_CICFlowMeter.csv`) and place them in `data/cic2018/` named `wed14.csv`, `thu15.csv`, `fri16.csv` and `wed21.csv`.
2. Preprocess, run and aggregate:

```bash
python src/prep_cic2018.py     # writes data/cic2018/cic2018_subset.csv and prep_log.json
python src/run_cic2018.py
python src/aggregate_cic2018.py
```

### Notes

- The run scripts execute 5 seeds (0–4) sequentially by default. Set the `SEEDS` environment variable to run a single seed, e.g. `SEEDS=2 python src/run_unridd.py`; single-seed runs write `results_s<n>.json`, and the default multi-seed run writes `results_all.json`. The aggregation scripts read every `results_*.json` in the output directory.
- Outputs are written to `results/<dataset>/` and `figures/<dataset>/` by default. Running the scripts **overwrites the pre-computed files**; to keep them, redirect outputs with `OUT_DIR=<dir>` (and `FIG_DIR=<dir>` for aggregation). `DATA_DIR` overrides the dataset location.
- On the reference machine the summed model training time was approximately 6–7 minutes per seed on UNR-IDD and 13–21 minutes per seed on CIC-IDS2018 (all seven models); wall-clock time including preprocessing and evaluation is longer and hardware-dependent.

## Pre-computed results

The `results/` directory contains the outputs of the reported experiments, so the paper's tables and figures can be inspected or regenerated without re-training:

- `results_s<n>.json` – per-seed metrics for every model (accuracy, weighted/macro precision, recall and F1, macro AUC, training time, inference latency) together with run metadata.
- `preds_seed<n>.npz` – per-seed test labels, class-probability arrays and autoencoder reconstruction errors.
- `summary.json` / `results.md` – 5-seed mean ± standard deviation.

Regenerating the summary tables and figures from these files takes seconds:

```bash
python src/aggregate_unridd.py
python src/aggregate_cic2018.py
```

```python
import numpy as np, json
z = np.load("results/unridd/preds_seed0.npz")
print(z.files)                                   # available arrays
print(json.load(open("results/unridd/summary.json"))["HBN+AE"]["acc"])   # [mean, std]
```

The models are trained and evaluated in memory; no serialised model weights (`.h5`/`.keras`/`.pkl`) were produced in the reported runs, so `models/` is empty. The JSON and NPZ files, together with fixed seeds, constitute the reproducibility artifacts. Minor numerical differences may arise across hardware and library builds (e.g. oneDNN floating-point ordering).

## Citation

[To be added after publication]

## License

This project is released under the MIT License – see [LICENSE](LICENSE).
