# Data

The inputs are hosted on Kaggle and are **not stored in this repository**. This page says which dataset is used for which study and how to put the files in place.

## Datasets

| Kaggle dataset | Files used | Used for |
|---|---|---|
| [`poorvie05/2023-data`](https://www.kaggle.com/datasets/poorvie05/2023-data) | `ANL-ALCF-DJC-THETA_20230101_20231231.csv`<br>`ANL-ALCF-DARSHAN-THETA_20230101_20231231.csv` | **Study A**, Theta runtime and Theta I/O |
| [`poorvie05/polaris-2025-data`](https://www.kaggle.com/datasets/poorvie05/polaris-2025-data) | `ANL-ALCF-DJC-POLARIS_20250101_20251231.csv`<br>`features_coarse.csv`<br>`features_fine.csv` | **Study A**, Polaris runtime and I/O (job log + `features_coarse.csv`)<br>**Study B**, Polaris early I/O prediction (all three files) |
| [`poorvie05/param-2025-data`](https://www.kaggle.com/datasets/poorvie05/param-2025-data) | `JobLog_Year_2025_time_fields_added.xlsx` | **Study A**, Param runtime |

### What each file contains

- **`ANL-ALCF-DJC-*`** – the scheduler job log, one row per job: user, project, queue, nodes requested, requested walltime, queued, start and end times, and run time. These are the request and history features for Study A.
- **`ANL-ALCF-DARSHAN-THETA_*`** – the Darshan I/O log for Theta jobs (processes, run time, bytes read and written). Used to derive the Theta I/O time target.
- **`features_coarse.csv`** – one row per Polaris job with its Darshan summary (nodes, processes, interface, total bytes).
- **`features_fine.csv`** – the same Polaris jobs with the 100 read and 100 write byte bins (`read_0 … read_99`, `write_0 … write_99`) that Study B reveals gradually.
- **`JobLog_Year_2025_time_fields_added.xlsx`** – the Param Slurm job log (`sacct`-style fields such as `JobID`, `User`, `Group`, `JobName`, `Submit`, `Start`, `End`, `Timelimit`, `ElapsedRaw`, `AllocCPUS`, `AllocNodes`, `State`, `Partition`). The first row of the sheet is a placeholder (`Column1…Column25`); the real header is the second row. Param has **no Darshan data**, so only its runtime is predicted.

## Which notebook reads what

| Notebook | Setting | Input files |
|---|---|---|
| `01_study_a_history_prediction.ipynb` | `MACHINE = 'theta'` | `2023-data`: Theta DJC + Theta Darshan |
| | `MACHINE = 'polaris'` | `polaris-2025-data`: Polaris DJC + `features_coarse.csv` |
| | `MACHINE = 'slurm'` (Param) | `param-2025-data`: the `.xlsx` |
| | `TARGET = 'RUNTIME_SECONDS'` or `'IO_TIME'` | Param supports runtime only |
| `02_study_b_early_io_prediction.ipynb` | – | `polaris-2025-data`: Polaris DJC, `features_coarse.csv`, `features_fine.csv` |

The five Study A runs are Theta runtime, Theta I/O, Polaris runtime, Polaris I/O and Param runtime.

## Getting the data

**On Kaggle (as the notebooks were run).** Attach the three datasets to the notebook as inputs. The default paths are:

```
/kaggle/input/datasets/poorvie05/2023-data/
/kaggle/input/datasets/poorvie05/polaris-2025-data/
/kaggle/input/datasets/poorvie05/param-2025-data/
```

**Locally.** Download with the Kaggle CLI (`pip install kaggle`, with an API token in `~/.kaggle/kaggle.json`):

```bash
kaggle datasets download -d poorvie05/2023-data         -p data/theta_2023   --unzip
kaggle datasets download -d poorvie05/polaris-2025-data -p data/polaris_2025 --unzip
kaggle datasets download -d poorvie05/param-2025-data   -p data/param_2025   --unzip
```

Then edit the path variables at the top of each notebook (`JOBLOG`, `darshan_path`, `jobcomp_path`, `COARSE_FILE`, `FINE_FILE`, `LOG_FILE`) to point at these folders. The expected layout is:

```
data/
├── theta_2023/
│   ├── ANL-ALCF-DJC-THETA_20230101_20231231.csv
│   └── ANL-ALCF-DARSHAN-THETA_20230101_20231231.csv
├── polaris_2025/
│   ├── ANL-ALCF-DJC-POLARIS_20250101_20251231.csv
│   ├── features_coarse.csv
│   └── features_fine.csv
└── param_2025/
    └── JobLog_Year_2025_time_fields_added.xlsx
```

## Source and citation

The Theta and Polaris logs are job and Darshan records from the Argonne Leadership Computing Facility (ALCF). The Param log comes from the Param supercomputer's Slurm accounting. Add the official citation or data-release link for each source here before publishing.
