# Whale coda dynamics

Sperm whales communicate with codas: short, stereotyped patterns of clicks, often exchanged between two whales. Neural sequence models predict a whale's next coda better when they see the order of up to eight preceding codas (Sharma et al., 2024b), and features of one whale's next coda can be predicted from its partner's preceding codas (Assayag et al., 2026). Terwilliger et al. (2025) caution that without timing information, such models may mistake persistence, such as a whale repeating a signal while waiting for a response, for compositional structure. This project asks where the predictability comes from: a whale's own earlier codas, its partner's codas, or the timing of the exchange.

Group project for *Multilingual NLP* (UvA MSc AI, 2026) by Ardjano Mark, Sara Diaz de la Fuente, Iulia Costea, Vlad Chibulcutean and Milan Tool.

## Approach

We treat this as next-token prediction. For each coda, a model predicts its type from some combination of three inputs:

- **A**, the whale's own preceding codas;
- **B**, its partner's preceding codas;
- **S**, the state of the exchange when the coda starts (whose turn it is, and how long the gap or overlap was).

We train a separate model for each of the eight combinations. The difference in held-out log-likelihood between two of them, in bits per coda, measures how much one input adds once the others are known. Our main comparison is $\mathrm{LL}(A,B,S) - \mathrm{LL}(B,S)$: what a whale's own history adds beyond its partner and the timing. Models are tested on one recording day at a time (14 days), and confidence intervals come from resampling days.

Two further questions build on this. RQ2 checks whether the conclusions depend on how codas are labelled, using seven labelling schemes. RQ3 asks how well a model of one clan's click rhythms transfers to other clans.

Exact commands and the order of runs are in [`docs/REPRODUCING.md`](docs/REPRODUCING.md).

## What we found

Values are bits per coda, with 95% confidence intervals over recording days.

- **Own history carries almost all the predictable information:** +1.34 [0.94, 1.73], positive on all 14 test days. Most of it comes from repetition: 82.5% of the 1,547 target codas repeat the whale's previous coda type. On codas where the whale switches type, own history adds about 0.13 bits.
- **The partner's codas and the timing added nothing we could detect** once own history was known: −0.03 [−0.09, 0.06] and −0.02 [−0.05, 0.00].
- Looking back further than three codas did not help (+0.01 [−0.01, 0.04]). A neural model stacked on the logistic baseline found nothing extra either.
- When whales switch type, they often return to the type they just left (47.6% of switches, against 26.8% by chance). Features for run length, recent distinct types and repertoire improve prediction slightly (+0.05 [0.02, 0.10]), but the order of earlier types adds little beyond the repertoire (+0.01, $p$ = 0.053), and the gain shrinks when ornamented codas are merged into their neighbour's type (+0.04, $p$ = 0.07). This is not clear evidence of switching structure beyond ornaments.
- Two follow-ups not covered in the report, with coda duration as the target or previous durations as extra inputs, found no detectable partner effect either.
- The pattern holds under all seven labelling schemes. Own history explains a smaller share of what is predictable when tempo is part of the label.
- Across nine clans, a model trained on another clan always predicts a clan's codas worse than its own model does (0.3 to 2.8 bits per timing symbol for n-gram models), partly because many timing patterns never occur in the other clan's data. Clans whose ranges overlap more did not transfer better. Pooling clans into one LSTM helps little overall, but telling the model which clan a coda comes from improves prediction for every clan.

These results describe short two-whale exchanges from one population's recordings. At this scale, a whale's own recent output is the best predictor of its next coda. That says nothing about grammar. And a null result for the partner means we could not detect an effect, not that there is none.

The main results, with tables, are in [`reports/README.md`](reports/README.md).

## Running the code

Python 3.10 or newer. Preprocessing uses only the standard library; the models need NumPy, SciPy and PyTorch (CPU is fine).

```sh
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install ".[modeling]" pytest
```

Run everything from the repository root with `PYTHONPATH=src` (PowerShell: `$env:PYTHONPATH = "src"`). Model runs used `OPENBLAS_NUM_THREADS=1 OMP_NUM_THREADS=1 MKL_NUM_THREADS=1`; set the same for matching numbers.

All model outputs are committed in `data/`, so nothing needs rerunning to check the results. To rebuild the data:

```sh
PYTHONPATH=src python -m whale_coda_dynamics.preprocessing.prepare          # RQ1 tables
PYTHONPATH=src python -m whale_coda_dynamics.preprocessing.folds            # 14 leave-one-day-out folds
PYTHONPATH=src python -m whale_coda_dynamics.preprocessing.inventories      # RQ2 labelling schemes
PYTHONPATH=src python -m whale_coda_dynamics.rq3.preprocessing.prepare --config config/preprocessing_rq3.json
```

Each experiment is one command, `python -m whale_coda_dynamics.<module> --config <config>`:

| Experiment | Module | Config |
|---|---|---|
| Main lattice | `logistic` | `rq1_logistic_v4.json` |
| Robustness check | `logistic_v5` | `rq1_logistic_v5.json` |
| Neural model on top | `residual` | `rq1_lstm_residual.json` |
| History length | `history_length` | `rq1_history_length.json` |
| Switch structure | `switch_structure` | `rq1_switch_structure.json` |
| Ornament merge | `ornament_merge` (see docs) | `rq1_ornament_merge.json` |
| Duration as target | `duration_lattice` | `rq1_duration_lattice.json` |
| Labelling schemes | `logistic_v5`, `rq2_compare` | `rq2_*_logistic_v5.json`, `rq2_compare_v5.json` |
| Partner tempo | `tempo_factor` | `rq2_partner_tempo.json` |
| Durations as input | `rubato` | `rq2_rubato_inputs.json` |
| Cross-clan transfer | `rq3.experiments.runner` | `rq3_experiments.json` |
| Pooled LSTM | `rq3_neural.runner --stage all` | `rq3_neural.json` |

Configs live in `config/`. Run order, stages and run times are in [`docs/REPRODUCING.md`](docs/REPRODUCING.md).

Tests (unit tests plus a small end-to-end run of each model runner):

```sh
PYTHONPATH=src python -m pytest tests
```

## Notes

- This repository is a snapshot of our development repository. Commit hashes inside `data/results/**/summary.json` refer to that repository.
- Older model versions (logistic v1–v3, the first LSTMs) are kept with their results, because later runs compare against them.
- Reruns on another machine agree on log-likelihoods to about $10^{-6}$; individual probabilities can differ slightly more.

## Data

The files in `data/raw/` come from other researchers and are included unchanged so the results can be reproduced. Field meanings and provenance are in [`docs/data_dictionary.md`](docs/data_dictionary.md).

| File | Source |
|---|---|
| `dswp-2014-2016-retagged-maor-2025.csv` | Assayag et al. (2026), [dataset and code](https://github.com/MaorAssayag/coda-sequences-relations-and-prediction); recordings by the Dominica Sperm Whale Project |
| `sperm-whale-dialogues.csv`, `rhythms.p`, `mean_codas.p`, `tempos-dict.p`, `ornaments.p`, `CarribeanCodas.csv` | Sharma et al. (2024a), [sw-combinatoriality](https://github.com/pratyushasharma/sw-combinatoriality) `data/`; `CarribeanCodas.csv` is their `DominicaCodas.csv`, renamed. Contents identical, line endings normalised to LF |
| `PacificCodas.csv` | Hersh et al. (2022), OSF `ae6pd` |

## References

- Assayag, M., Gero, S., Paradise, O., & Diamant, R. (2026). Exploring statistical relations between coda sequences of Dominica sperm whale pairs. *Bioacoustics*, 35(4), 472–500. https://doi.org/10.1080/09524622.2026.2684665
- Hersh, T. A., Gero, S., Rendell, L., et al. (2022). Evidence from sperm whale clans of symbolic marking in non-human cultures. *Proceedings of the National Academy of Sciences*, 119(37), e2201692119. https://doi.org/10.1073/pnas.2201692119
- Sharma, P., Gero, S., Payne, R., Gruber, D. F., Rus, D., Torralba, A., & Andreas, J. (2024a). Contextual and combinatorial structure in sperm whale vocalisations. *Nature Communications*, 15, 3617. https://doi.org/10.1038/s41467-024-47221-8
- Sharma, P., Gero, S., Rus, D., Torralba, A., & Andreas, J. (2024b). WhaleLM: Finding structure and information in sperm whale vocalizations and behavior with machine learning. *bioRxiv*. https://doi.org/10.1101/2024.10.31.621071
- Terwilliger, J., Kaufhold, S. P., & Rossano, F. (2025). A functionalist framework for machine learning in animal communication. *NeurIPS 2025 Workshop: AI for Non-Human Animal Communication*.

## Use of generative AI
We used generative AI (Claude, Codex) as a coding assistant: to help write and maintain the codebase, run the experiments, and draft documentation such as this README.All research questions, design decisions and interpretation of the results are ours, and we reviewed all code and outputs.
