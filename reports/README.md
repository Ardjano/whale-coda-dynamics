# Results

Main results for all three research questions. Log-likelihoods are held-out bits per coda (RQ1, RQ2) or per timing symbol (RQ3). Intervals are 95% bootstrap intervals over recording days (RQ1, RQ2) or recording groups (RQ3). "Days" counts the 14 test days on which a contrast is positive, and $p$ is an exact two-sided sign-flip test over days. Rerunning an experiment (see [`docs/REPRODUCING.md`](../docs/REPRODUCING.md)) writes its full report to this folder.

## RQ1: own history, partner history and timing

Logistic regression, 1,547 targets, 14 leave-one-day-out folds.

| Inputs | LL (bits/coda) | Accuracy |
|---|---:|---:|
| none (unigram) | −2.603 | 57.2% |
| $S$ | −2.554 | 57.3% |
| $A$ | −1.097 | 82.4% |
| $B$ | −2.473 | 59.0% |
| $A, B$ | −1.112 | 82.5% |
| $A, S$ | −1.107 | 82.2% |
| $B, S$ | −2.476 | 58.0% |
| $A, B, S$ | −1.133 | 82.3% |
| repeat own last coda (baseline) | −1.393 | 82.5% |

| Contrast | Gain | 95% CI | Days | Repeat part | Switch part |
|---|---:|---|---:|---:|---:|
| Own history, $\mathrm{LL}(A,B,S) - \mathrm{LL}(B,S)$ | +1.344 | [0.945, 1.732] | 14/14 | 1.210 | 0.134 |
| Partner history, $\mathrm{LL}(A,B,S) - \mathrm{LL}(A,S)$ | −0.026 | [−0.094, 0.064] | 6/14 | | |
| Timing, $\mathrm{LL}(A,B,S) - \mathrm{LL}(A,B)$ | −0.020 | [−0.046, 0.001] | 7/14 | | |

82.5% of targets repeat the whale's previous coda type. The repeat part is the gain on predicting whether the whale repeats; the switch part is the gain on predicting which type it switches to.

| Robustness check and follow-up | Result |
|---|---|
| Per-fold selection of input blocks (logistic v5) | own +1.340 [0.939, 1.725], partner −0.043 [−0.102, 0.010], timing −0.018 [−0.045, 0.004] |
| Residual LSTM on top of the logistic model | no condition gains: every interval spans zero (e.g. $A,B,S$: −0.024 [−0.104, 0.015]) |
| Nine instead of three own codas, in $A,B,S$ | +0.009 [−0.014, 0.037], $p$ = 0.98; a planted lag-6 pattern worth > 0.30 bits/coda is detected |
| Switch-backs to the type just left | 47.6% of switches, against 26.8% expected by chance |
| Run length, recent distinct types and repertoire, all together | +0.052 [0.023, 0.099], 13/14 days, $p$ = 0.008 |
| Order of earlier types beyond the repertoire | +0.013 [0.002, 0.027], 10/14 days, $p$ = 0.053 |
| The same features with ornamented codas merged into their neighbour's type | +0.036 [0.003, 0.078], 12/14 days, $p$ = 0.073 |
| Coda duration as the target: partner history | +0.026 [−0.064, 0.098], $p$ = 0.58 (own history +0.926 [0.742, 1.163]) |

## RQ2: alternative coda labellings

The same contrasts on the same 1,547 targets under seven labellings. A: the source's named types (RQ1). B: a second classifier's types. C: Sharma et al.'s rhythm templates. D and E: rhythm or named type crossed with a tempo class; D′ and E′: the same with alternative tempo boundaries. Log-likelihoods are not comparable across labellings, so the second table expresses each contrast as a share of the labelling's total predictability, $\mathrm{LL}(A,B,S) - \mathrm{LL}(\text{none})$.

| Labelling | Own history | Partner | Timing |
|---|---|---|---|
| A | +1.340 [0.939, 1.725] | −0.043 [−0.102, 0.010] | −0.018 [−0.045, 0.004] |
| B | +1.507 [0.993, 2.024] | −0.063 [−0.144, 0.008] | −0.013 [−0.026, 0.000] |
| C | +0.868 [0.550, 1.233] | −0.037 [−0.091, 0.010] | −0.004 [−0.017, 0.005] |
| D | +1.439 [1.138, 1.719] | −0.005 [−0.018, 0.005] | −0.003 [−0.006, −0.001] ($p$ = 0.06) |
| E | +1.485 [1.145, 1.813] | −0.060 [−0.148, 0.022] | +0.000 [−0.003, 0.005] |
| D′ | +1.488 [1.185, 1.760] | +0.001 [−0.011, 0.017] | −0.002 [−0.008, 0.008] |
| E′ | +1.472 [1.130, 1.782] | −0.046 [−0.121, 0.027] | −0.004 [−0.009, 0.001] |

| Labelling | Own-history share | Partner share | Timing share | Own share − A |
|---|---|---|---|---|
| A | 0.922 [0.641, 1.213] | −0.029 [−0.073, 0.006] | −0.013 [−0.032, 0.002] | |
| B | 1.036 [0.696, 1.434] | −0.043 [−0.104, 0.005] | −0.009 [−0.019, 0.000] | +0.113 [0.008, 0.236] |
| C | 0.961 [0.726, 1.193] | −0.041 [−0.090, 0.011] | −0.004 [−0.021, 0.005] | +0.039 [−0.086, 0.228] |
| D | 0.728 [0.565, 0.878] | −0.003 [−0.010, 0.003] | −0.001 [−0.003, −0.000] | −0.194 [−0.367, −0.002] |
| E | 0.757 [0.564, 0.966] | −0.031 [−0.079, 0.010] | 0.000 [−0.001, 0.003] | −0.165 [−0.314, −0.009] |
| D′ | 0.740 [0.584, 0.883] | 0.001 [−0.006, 0.009] | −0.001 [−0.004, 0.004] | −0.183 [−0.364, 0.013] |
| E′ | 0.731 [0.549, 0.920] | −0.023 [−0.063, 0.013] | −0.002 [−0.004, 0.000] | −0.191 [−0.341, −0.038] |

Shares can exceed 1 because the partner and timing contrasts are slightly negative. Two further tests: whether the partner's codas predict a coda's tempo class found nothing (all four contrasts span zero, in both versions of the test), and adding the previous codas' durations as inputs gave a partner contrast of −0.055 [−0.106, 0.021] on D ($p$ = 0.21).

## RQ3: cross-clan transfer

Transfer loss = own-clan log-likelihood minus the source clan's log-likelihood on the target clan's held-out codas, in bits per timing symbol (50-ms inter-click-interval bins). Positive means the source model predicts worse. Smoothed n-gram models:

| Source \ Target | EC1 | EC2 | FP | PALI | PO | REG | RI | SH | SI |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| EC1 | 0 | 1.70 | 1.50 | 1.48 | 1.96 | 1.53 | 1.25 | 1.53 | 2.65 |
| EC2 | 2.32 | 0 | 1.50 | 2.12 | 1.66 | 1.07 | 1.73 | 2.00 | 2.45 |
| FP | 1.95 | 1.07 | 0 | 1.26 | 0.97 | 0.89 | 0.63 | 0.68 | 1.47 |
| PALI | 2.07 | 1.97 | 1.32 | 0 | 1.89 | 1.25 | 0.61 | 1.44 | 2.22 |
| PO | 2.82 | 1.12 | 1.30 | 2.04 | 0 | 1.00 | 2.07 | 1.58 | 2.07 |
| REG | 2.11 | 0.41 | 1.49 | 1.53 | 1.31 | 0 | 1.37 | 1.54 | 2.42 |
| RI | 1.63 | 1.49 | 0.78 | 0.72 | 1.33 | 1.12 | 0 | 0.51 | 1.54 |
| SH | 1.64 | 1.12 | 0.83 | 0.76 | 1.03 | 0.87 | 0.32 | 0 | 1.39 |
| SI | 2.80 | 0.79 | 1.49 | 1.69 | 1.77 | 1.22 | 1.44 | 1.14 | 0 |

![Transfer loss matrix](rq3_transfer_heatmap.png)

All 72 off-diagonal losses are positive, and every interval is above zero. A target coda's complete timing sequence occurs at least five times in the source clan's training data for only 1.9–12.9% of codas, and for every target clan uncovered codas have larger losses than covered ones. In the Pacific data, pairs of clans whose ranges overlap more did not transfer better (Pearson $r$ = 0.35 for identity codas, 0.56 for non-identity codas, both positive).

LSTMs, per target clan (bits per timing symbol; the last four columns are differences, positive favours the first-named model):

| Clan | Own LSTM | n-gram | Pooled − own | Clan tag − none | Leave-one-out − mean single source | Fine-tuned − scratch (2% of data) |
|---|---:|---:|---|---|---|---|
| EC1 | −1.553 | −1.659 | −0.016 [−0.027, −0.005] | +0.329 [0.305, 0.352] | +0.652 [0.623, 0.685] | +0.161 [0.124, 0.190] |
| EC2 | −1.509 | −1.630 | +0.085 [0.029, 0.196] | +0.451 [0.306, 0.539] | +0.650 [0.598, 0.740] | +0.868 [0.690, 0.998] |
| FP | −2.726 | −2.871 | +0.053 [0.007, 0.107] | +0.356 [0.234, 0.450] | +0.831 [0.749, 0.921] | +0.259 [0.165, 0.371] |
| PALI | −1.807 | −2.004 | +0.000 [−0.013, 0.015] | +0.427 [0.382, 0.468] | +0.517 [0.478, 0.553] | +0.103 [0.041, 0.171] |
| PO | −1.708 | −1.848 | +0.009 [−0.023, 0.041] | +0.445 [0.368, 0.526] | +0.472 [0.460, 0.485] | +0.259 [0.132, 0.396] |
| REG | −1.363 | −1.444 | −0.005 [−0.013, 0.008] | +0.188 [0.139, 0.233] | +0.543 [0.503, 0.605] | +0.231 [0.150, 0.297] |
| RI | −2.124 | −2.222 | +0.024 [−0.003, 0.050] | +0.247 [0.091, 0.414] | +0.796 [0.744, 0.851] | +0.305 [0.181, 0.433] |
| SH | −2.283 | −2.356 | −0.015 [−0.040, 0.011] | +0.288 [0.198, 0.367] | +0.723 [0.674, 0.763] | +0.223 [0.123, 0.316] |
| SI | −1.853 | −1.956 | −0.008 [−0.040, 0.029] | +0.588 [0.470, 0.689] | +0.745 [0.657, 0.863] | +0.374 [0.268, 0.518] |

Figures: [`rq3_neural_figures/`](rq3_neural_figures/).
