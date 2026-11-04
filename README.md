# OpenKeystroke

An open research platform for **authorized acoustic side-channel experiments and privacy-defense evaluation**.
It classifies predefined, non-sensitive acoustic categories in controlled or explicitly consented laboratory data,
and measures how signal transformations change how distinguishable those categories are.

It deliberately does **not** reconstruct text, passwords or messages, capture audio covertly, or ingest recordings it was
not explicitly given with a consent record. Recordings stay local; nothing is uploaded.

## Install

```bash
pip install -e ".[dev]"
openkeystroke doctor --self-test
```

## Quick start

```bash
openkeystroke dataset generate --out data/syn           # deterministic SYNTHETIC data
openkeystroke dataset validate data/syn
openkeystroke experiment run --preset model_comparison   # dataset -> features -> train -> evaluate -> artifacts
openkeystroke experiment list
openkeystroke experiment inspect <id>
openkeystroke benchmark run --preset defense_comparison  # static vs adaptive attackers, distortion, Pareto set
openkeystroke experiment reproduce <id> --tol 1e-6       # re-run from the immutable config and diff every metric
openkeystroke dashboard                                  # Streamlit workspace reading the same registry
```

Python API: `from openkeystroke import preset_config, run_experiment, Workspace, get_summary`.
One configuration schema (`ExperimentConfig`, YAML/JSON) is shared by the CLI, the Python API and the dashboard.

## Validity rules enforced by the code

* Splits are reproducible; `session_disjoint`, `device_disjoint` and `environment_disjoint` protocols guarantee group-disjoint
  train/val/test, and every run is leak-checked (indices, groups, identical waveforms) before training.
* Standardisation, PCA and normalisation statistics are fit on the training partition only; hyper-parameter / variant selection
  uses the validation partition; reported metrics are held-out test metrics.
* Confidence intervals use a cluster bootstrap over sessions; seed-to-seed spread uses t-intervals; comparisons are paired by seed.
* A defense is only called effective if it shows a statistically significant accuracy drop (>= 3 seeds) against **both** a static
  attacker and an adaptive attacker retrained on defended audio.
* Synthetic datasets are labelled everywhere; synthetic results are not evidence of real-world performance.
* Pickle is never loaded from unverified files; model checkpoints use `torch.load(weights_only=True)`.

## Layout

`src/openkeystroke/`: `data/` (schema, synthetic generator, consent ingestion, splits, leakage), `dsp/`, `features/`,
`models/` (classical + PyTorch, trainer), `evaluation/`, `defense/`, `registry/` (SQLite + artifact store),
`experiments/` (plan, runner, bounded local executor), `dashboard/`, `cli.py`, `api.py`.
