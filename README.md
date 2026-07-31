# Modeling Recorded Reality: Informative-Missingness-Aware Generation of Clinical Time Series

Anonymized code release for the submission of the same title. This repository contains the
model (**RecordDiff**), the two evaluation methodologies (channel-decomposition transfer and
the dual-channel privacy audit), and the notebooks that reproduce every table and figure in
the paper.

> **Anonymity.** This repository is anonymized for double-blind review. It contains no author,
> institution, or affiliation information. Please do not attempt to de-anonymize it.

---

## 1. Overview

RecordDiff models the **recorded reality** of an EHR, the joint distribution of the measurement
mask and the recorded values, rather than imputing missingness away. It couples a Bernoulli
measurement policy (Stage 1) and a conditional value-diffusion decoder (Stage 2) through a
shared latent patient-state trajectory, trained end to end by amortized variational inference.
Beyond the model, we contribute two reusable evaluation tools: a channel-decomposition transfer
protocol that isolates whether synthetic data preserves the information carried by the
measurement pattern, and a dual-channel privacy audit that treats the measurement pattern as a
quasi-identifier.

---

## 2. Data (not included)

The paper uses two **credentialed-access** datasets from PhysioNet:

- **MIMIC-IV** (primary cohort, 36 variables, 48-hour ICU windows)
- **eICU Collaborative Research Database** (multi-center generalization cohort, 35 variables)

**No data, raw or derived, is included in this repository**, in compliance with the PhysioNet
Data Use Agreement. To reproduce results you must obtain the datasets yourself: complete the
required credentialing on PhysioNet, download MIMIC-IV and eICU, then point the notebooks at
your local copy.

**Preprocessing is built into the two audit notebooks.** `Mask_Channel_Audit_MIMIC_IV.ipynb`
and `Mask_Channel_Audit_eICU.ipynb` each construct their cohort from the raw PhysioNet tables
(cohort selection, the 48-hour window, and the `(N, T, V)` mask and value tensors) and cache the
result. Every other notebook consumes those cached cohorts. Run the two audit notebooks first.

Set your paths once, at the top of each notebook or via environment variables, and do not commit
anything under them:

```bash
export RECORDDIFF_DATA_DIR=/path/to/your/physionet/data     # raw MIMIC-IV / eICU
export RECORDDIFF_OUT_DIR=/path/to/write/cohorts/and/results
```

---

## 3. Environment

Python 3.10+. Install dependencies with:

```bash
pip install -r requirements.txt
```

Core dependencies: `torch`, `numpy`, `scipy`, `scikit-learn`, `pandas`, `matplotlib`, `tqdm`.
The notebooks were developed in a GPU runtime; a single modern GPU is sufficient for model
training. The audits and the controlled experiment run on CPU, though more slowly.

---

## 4. Repository structure

```
Data preparation + mask-channel audit  (these build and cache the cohorts)
  Mask_Channel_Audit_MIMIC_IV.ipynb   Build MIMIC-IV cohort; model-free mask-channel audit (Table 1)
  Mask_Channel_Audit_eICU.ipynb       Build eICU cohort; model-free mask-channel audit (eICU)

Model
  RecordDiff_Model_Train.ipynb        RecordDiff: state-space latent, Bernoulli measurement policy,
                                      conditional value-diffusion decoder, amortized-VI encoder,
                                      per-variable Gaussianization, three-phase training

Controlled and semi-synthetic validation
  Controlled_Experiment.ipynb         Fully synthetic MNAR sweep; oracle vs learned policy vs
                                      value-blind baselines (controlled-sweep figure)
  SemiSynthetic_recovery.ipynb        Semi-synthetic recovery of an injected policy (rank corr. 0.96)

Real-data evaluation
  Fidelity.ipynb                      Recorded-law fidelity (mask density, co-occurrence, MMD,
                                      value KS); PCA overlap figure
  RecordDiff_TSTR_mimic.ipynb         Channel-decomposition transfer, MIMIC-IV (RecordDiff, TRTR
                                      ceiling, mask-agnostic ablation)
  RecordDiff_TSTR_eICU.ipynb          Channel-decomposition transfer, eICU
  Mechanism_&Privacy.ipynb            Mechanism plausibility (MAR-restricted variant) and the
                                      dual-channel privacy audit (Table 3; DCR figure)

Baseline (TimeDiff, via the interleaving adapter)
  TimeDiff_TSTR_mimic.ipynb           TimeDiff channel-decomposition transfer, MIMIC-IV
  TimeDiff_TSTR_eICU.ipynb            TimeDiff channel-decomposition transfer, eICU

Support
  requirements.txt
  README.md
```

**Ablations and variants are configurations, not separate files.** The mask-agnostic ablation is
produced in the `RecordDiff_TSTR_*` notebooks; the independent-marginal-mask and mask-replay
generators are configurations in `Controlled_Experiment.ipynb`; the MAR-restricted variant is in
`Mechanism_&Privacy.ipynb`.

---

## 5. Reproducing the paper

Suggested run order: build the cohorts (the two audit notebooks), train the model, then run the
evaluation notebooks. `Controlled_Experiment.ipynb` is fully synthetic and self-contained, so it
can be run on its own without any PhysioNet data.

| Paper item | Notebook(s) |
|---|---|
| Table 1 (mask-channel audit, MIMIC-IV) | `Mask_Channel_Audit_MIMIC_IV.ipynb` |
| Mask-channel audit, eICU | `Mask_Channel_Audit_eICU.ipynb` |
| Controlled MNAR sweep (figure) | `Controlled_Experiment.ipynb` |
| Semi-synthetic recovery (rank corr. 0.96) | `SemiSynthetic_recovery.ipynb` |
| Recorded-law fidelity + PCA figure | `Fidelity.ipynb` |
| Table 2 (channel-decomposition transfer) | `RecordDiff_TSTR_mimic.ipynb`, `RecordDiff_TSTR_eICU.ipynb`, `TimeDiff_TSTR_mimic.ipynb`, `TimeDiff_TSTR_eICU.ipynb` |
| Table 3 (mechanism plausibility + privacy) + DCR figure | `Mechanism_&Privacy.ipynb` |
| RecordDiff model (used by the evaluation notebooks) | `RecordDiff_Model_Train.ipynb` |

Table 2 combines the four transfer notebooks: the `RecordDiff_TSTR_*` notebooks provide the
RecordDiff, TRTR-ceiling, and mask-agnostic columns, and the `TimeDiff_TSTR_*` notebooks provide
the released-baseline column.

---

## 6. Reproducibility notes

- **Seeds.** Stochastic steps are seeded at the top of each notebook. The controlled experiment
  fixes and reports a set of 20 seeds.
- **Uncertainty.** Transfer results report confidence intervals; the controlled experiment
  reports bootstrap confidence intervals over the 20 seeds.
- **Significance.** The controlled experiment assesses significance with sign-flip randomization
  under Benjamini-Hochberg false-discovery-rate control.
- **Baseline budget.** TimeDiff is trained for a reduced step budget relative to its default
  schedule, as documented in the paper and in the `TimeDiff_TSTR_*` notebooks. The mask-channel
  result, which carries the paper's claim, is unaffected by this budget.
