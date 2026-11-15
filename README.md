# Predicting HPC Job Runtime and I/O Time from History and Early Darshan Signals

Poorvie Sadagopan · Mentor: Prof. Preeti Malakar

Two machine-learning studies on supercomputer job logs.

| Study | Question | Data |
|---|---|---|
| **A: before the job runs** | Can runtime and I/O time be predicted from the job's request and the history of earlier finished jobs? | Theta (runtime, I/O), Polaris (runtime, I/O), Param (runtime) |
| **B: while the job runs** | How early can the I/O time of a running job be predicted from the first *k* of 100 Darshan time bins? | Polaris, 4,807 jobs |

## Headline results

Test MdAPE of tuned CatBoost (Study A), time-ordered 80/20 split:

| Run | MdAPE | MAE (log units) |
|---|---|---|
| Theta runtime | 13.9% | 0.30 |
| Theta I/O | 39.2% | 0.76 |
| Polaris runtime | 31.3% | 0.41 |
| Polaris I/O | 56.5% | 1.00 |
| Param runtime | 28.1% | 0.87 |

Study B (Polaris I/O): MdAPE falls from **77.1%** with no bins to **44.5%** with the first 50 of 100 bins (MAE 1.27 → 0.79).

Main findings: accuracy depends on how often a job has an exact-match history (92% of Theta test jobs, 41% of Polaris); group history is the most useful input family; clustering and contention features add little; large jobs are harder; half of the Darshan trace gives most of the early-prediction benefit.

## Repository layout

- `notebooks/` – the two analysis notebooks
- `results/study_a/<run>/` – tables and figures for each of the five runs
- `results/study_b/` – tables and figures for the early-prediction study
- `data/` – inputs

## Metrics

All models predict `log(1 + time)`. Reported: **MdAPE** (median of |pred − actual| / (actual + 1)), **MAE** of `log(1 + time)` (log units: 0.7 ≈ a typical miss of ×2), and the share of jobs predicted **within 2×**.

## Reproducing the results

```bash
pip install -r requirements.txt
```

**Study A** (`notebooks/01_study_a_history_prediction.ipynb`)
1. Put the job log and Darshan files in `data/` and set the paths in the first cells.
2. Set `MACHINE` (`'theta'`, `'polaris'` or `'slurm'`, which is Param) and `TARGET` (`'RUNTIME_SECONDS'` or `'IO_TIME'`).
3. Run all cells. Tables and figures are written with the tag `<machine>_<runtime|io>`. Param has no I/O data, so only its runtime run exists.
4. Repeat for the five runs.

**Study B** (`notebooks/02_study_b_early_io_prediction.ipynb`)
1. Place the coarse Darshan file, the fine (binned) Darshan file and the job log in `data/` and set `COARSE_FILE`, `FINE_FILE` and `LOG_FILE`.
2. Run all cells. Outputs go to `./outputs`. Settings: `SEED = 42`, five seeds, 40 tuning trials, `K_GRID = (0, 1, 2, 5, 10, 25, 50, 75, 100)`.

**Report:** `cd report && pdflatex report.tex` (run twice).

## Method in brief

- **Study A:** features are the request plus the history of earlier finished jobs of the same group (user, project, walltime, nodes). When fewer than three such jobs exist, the group is widened step by step. 14 model variants are compared in direct and residual mode; tuned CatBoost is used as one fixed model. K-means clusters and contention features are tested on top.
- **Study B:** only the first *k* bins are used, with a leakage check (scrambling later bins must not change the features). Four feature sets (totals, shape, segments, raw) with optional context features, ten models, five-fold expanding-window CV. The operating point is the smallest *k* whose CV MAE is within 10% of the all-bins value (*k* = 50).

## Acknowledgements

I thank Prof. Preeti Malakar for her guidance and feedback. The Theta and Polaris logs are from the Argonne Leadership Computing Facility, and the Param log is from the Param supercomputer.

**Use of AI tools:** Claude (Anthropic) was used to help write and debug code, draft and edit the report and presentation text, create figures and format the report. All experiments were run by me, and I reviewed and take responsibility for the results and content.
