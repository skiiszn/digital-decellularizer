# Digital Decellularizer

Decellularization removes the cells from a tissue and leaves its extracellular-matrix
(ECM) scaffold behind, useful for regenerative medicine, but it's hard to predict which
proteins actually stay in that scaffold and which wash out.

The project uses placenta's native proteome and predicts **which proteins survive** decellularization and **how much their abundance shifts**.

The deliverable is a Jupyter Notebook that trains the model lineup, validates it and writes
a calibrated per-protein retention table (`results_retention_predictions.csv`).

## Headline results (5-fold CV)

| Question | Best model | Metric |
|---|---|---|
| How much does a protein shift? | **V10e** (flows, no ESM) | Pearson r = 0.648 ± 0.013 |
| Does a protein survive? | **V11e** (survival head, no ESM) | Δ-R² = 0.611 ± 0.009, AUROC = 0.875 |

Key finding: a protein language model (ESM-2) adds **no** measurable predictive value at this
dataset scale. The final models use no ESM and are smaller and faster as a result.

## Setup

Requires Python 3.10+.

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

A CUDA GPU is optional, the notebook auto-detects it and falls back to CPU.

## Data

Place these two files in a `data/` folder next to the notebook:

| File | Description |
|---|---|
| `data/placenta_annotated.csv` | Annotated placenta proteomics (provided with the project) |
| `data/uniprot_features_cache.csv` | UniProt feature cache (mol_weight, seq_length). Auto-generated on first run of the ESM utilities cell if absent. |

The ESM-2 embedding cell downloads `facebook/esm2_t6_8M_UR50D` on first run and caches it.
Because the final models do not use ESM, it can be skipped and still run V8e/V10e/V11e
and the actionable results.

## Run

Open the notebook and run **Kernel -> Restart & Run All**:

```bash
jupyter notebook digital_decellularizer.ipynb
```

Outputs written to the working directory:
- `results_retention_predictions.csv` — the actionable per-protein retention table
- `fig_final_summary.png` — final model comparison figure

## Configuration

All paths and run settings are in **one place**: the `GlobalConfig` dataclass in the Setup
cell (`data_path`, `cache_path`, `seed`, imputation parameters). There are no hardcoded
absolute paths. Edit that dataclass to point at data in a different location.

## Structure

- **Part I** — infrastructure, data loading, preprocessing ablation
- **Part II** — model ladder V1 -> V11 (baseline → uncertainty → flows → ESM → survival) + grid search
- **Part III** — calibration, actionable retention table, exploratory heart transfer, summary
