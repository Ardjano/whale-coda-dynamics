# Reproducing the results

All model outputs are committed in `data/`, and the main results are in [`reports/README.md`](../reports/README.md). Rerunning a step overwrites its files in `data/` (`git diff` shows what changed) and writes a detailed report to `reports/`, which git ignores.

## Setup

- Python 3.10 or newer: `pip install ".[modeling]" pytest` (NumPy, SciPy, PyTorch), plus `pip install matplotlib` for the RQ3 figures.
- Run from the repository root with `PYTHONPATH=src`.
- For model runs, set `OPENBLAS_NUM_THREADS=1 OMP_NUM_THREADS=1 MKL_NUM_THREADS=1`. The committed results were produced this way. Other settings change log-likelihoods by about $10^{-6}$.
- Hardware: preprocessing and the logistic models run in minutes on a laptop. The residual LSTM took about 2.3 hours on 18 CPU processes, and the pooled RQ3 LSTM about 1 hour on one RTX 3090.

In the commands below, `run <module> <config>` stands for

```sh
PYTHONPATH=src python -m whale_coda_dynamics.<module> --config config/<config>
```

Steps are listed in the order they depend on each other.

## 1. Preprocessing

```sh
run preprocessing.prepare        rq1.json                  # RQ1 tables
run preprocessing.folds          rq1_folds.json            # 14 leave-one-day-out folds
run preprocessing.folds          rq1_folds_retrospective.json
run preprocessing.prepare_rq2    rq2.json                  # same pipeline on Sharma et al.'s release (check only)
run rq3.preprocessing.prepare    preprocessing_rq3.json
run rq3.preprocessing.validate   preprocessing_rq3.json
run rq3.preprocessing.prepare    preprocessing_rq3_sensitivity_zzz.json
```

## 2. RQ1: own history, partner history and timing

```sh
run logistic          rq1_logistic_v4.json                 # main eight-condition lattice
run logistic          rq1_logistic_v4_retrospective.json   # sensitivity: alternative delay definition
run logistic_v5       rq1_logistic_v5.json                 # robustness check (needs v4)
run logistic_v5       rq1_logistic_v5_retrospective.json

# residual LSTM on top of logistic v4
run residual          rq1_lstm_residual_checks.json
run residual          rq1_lstm_residual_throughput.json
run residual          rq1_lstm_residual.json
run residual_planted  rq1_lstm_residual_planted.json

run history_length    rq1_history_length.json              # 1–9 own codas
run switch_structure  rq1_switch_structure.json
run duration_lattice  rq1_duration_lattice.json            # duration as target, ~30 s
```

`history_length` and `switch_structure` accept `--workers N` (default 4 and 5).

## 3. RQ2: alternative coda labellings

```sh
run sharma_inventory            sharma_inventory.json      # rhythm templates; output is committed
run preprocessing.inventories   rq2_inventories.json

for inv in b_dtree c_rhythm d_rhythm_tempo e_type_tempo d_rhythm_tempo_tdict e_type_tempo_tdict; do
  run preprocessing.folds  rq2_${inv}_folds.json
  run logistic             rq2_${inv}_logistic_v4.json
  run logistic_v5          rq2_${inv}_logistic_v5.json
done

run rq2_compare     rq2_compare.json        # comparison on v4
run rq2_compare     rq2_compare_v5.json     # comparison on v5 (reported)
run tempo_factor    rq2_partner_tempo.json
run rubato          rq2_rubato_inputs.json  # needs the v4 runs of RQ1, D and E′
```

Ornament merge (an extra inventory, O):

```sh
PYTHONPATH=src python -m whale_coda_dynamics.ornament_merge --stage build   --config config/rq2_o_ornament_merged.json
run preprocessing.folds  rq2_o_ornament_merged_folds.json
run logistic             rq2_o_ornament_merged_logistic_v4.json
run switch_structure     rq1_switch_structure_ornament_merged.json
PYTHONPATH=src python -m whale_coda_dynamics.ornament_merge --stage compare --config config/rq1_ornament_merge.json
```

## 4. RQ3: cross-clan transfer

```sh
for stage in calibration initial-transfer matrix bootstrap identity overlap summaries; do
  PYTHONPATH=src python -m whale_coda_dynamics.rq3.experiments.runner --config config/rq3_experiments.json --stage $stage
done

# sensitivity: EC2 codas with unit ZZZ excluded
for stage in calibration initial-transfer matrix bootstrap identity overlap summaries; do
  PYTHONPATH=src python -m whale_coda_dynamics.rq3.experiments.runner --config config/rq3_experiments_zzz_sensitivity.json --stage $stage
done
PYTHONPATH=src python -m whale_coda_dynamics.rq3.experiments.runner --config config/rq3_experiments_zzz_sensitivity.json --stage sensitivity-report

# pooled LSTM: stages pilot, arm1, arm23, capacity, arm4, arm5, report (each reads the earlier ones)
PYTHONPATH=src python -m whale_coda_dynamics.rq3_neural.runner --config config/rq3_neural.json --stage all
PYTHONPATH=src python -m whale_coda_dynamics.rq3_neural.plots  --config config/rq3_neural.json
```

## Tests

```sh
PYTHONPATH=src python -m pytest tests
```

Unit tests, leakage checks, and a small end-to-end run of each model runner on toy settings (a few minutes; the model tests are skipped without NumPy, SciPy and PyTorch).
