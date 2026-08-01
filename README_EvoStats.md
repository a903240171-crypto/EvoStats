# EvoStats Artifact

Artifact for:

> **EvoStats: Geometry-Aware Maintenance of Multivariate Statistics under Dependency Drift**  
> Bilong Wang, Zewen Lyu, and Xu Song  
> Submitted to PVLDB Volume 20 / VLDB 2027

EvoStats is an external PostgreSQL controller for auditing and maintaining stale multivariate statistics under dependency drift. The artifact supports verification of the paper's frozen results and, with PostgreSQL and the required ACS inputs, end-to-end reproduction of the principal experiments.

## Artifact scope

The artifact contains:

- source code for the controlled and real-data experiments;
- frozen outputs used for the paper;
- exact seeds and environment metadata;
- PostgreSQL experiment runners;
- graph-discovery, refresh-frontier, and action-aware planning code;
- claim-to-file provenance metadata;
- checksum validation utilities;
- quick verification scripts that do not require PostgreSQL;
- full runners for PostgreSQL-dependent experiments.

The paper studies:

1. catalog blindness under dependency drift;
2. geometry-dependent discovery cost;
3. refresh-frontier planning;
4. leave-one-hub-out robustness;
5. proactive auditing versus query feedback;
6. optional retire/refresh classification with a conservative multi-cell gate.

## Repository layout

```text
.
├── README.md
├── validate_artifact.py
├── reproduce_tables.py
├── CLAIMS_PROVENANCE.csv
├── MANIFEST.csv
├── code/
│   ├── phase5_validation/
│   └── scripts/experiments/
├── results/
│   ├── phase0_unified/
│   ├── phase1_statistic/
│   ├── phase2_v2_theory_aligned/
│   ├── phase3_pg_end_to_end_v2/
│   ├── phase4g_ambient_width/
│   ├── phase5c_frontier_audit/
│   ├── phase5f_d109_support_runtime/
│   ├── phase5g_query_feedback/
│   └── ...
└── environment/
```

Some directory names may differ slightly between the frozen submission release and the working repository. The frozen release is authoritative for paper claims.

## Quick verification

Quick verification checks the integrity of the release and reproduces combinatorial, classifier, and table-level claims from frozen outputs. It does not require PostgreSQL.

### 1. Validate checksums

```bash
python validate_artifact.py
```

Expected result:

```text
PASS: all checksums match.
```

### 2. Reproduce frozen claims and tables

```bash
python reproduce_tables.py
```

This step reconstructs paper-facing summaries from the frozen CSV/JSON outputs and checks the values recorded in `CLAIMS_PROVENANCE.csv`.

Representative expected claims include:

- 109 attributes and 5,886 candidate pairs;
- 141 severe pairs at support floor `m = 20`;
- 111 severe pairs at support floor `m = 100`;
- full support graph:
  - 141 edges,
  - 109 active columns,
  - maximum degree 108,
  - certified matching size 8;
- leave-one-hub-out graph:
  - 33 edges,
  - 27 active columns,
  - 4 connected components,
  - maximum degree 12,
  - certified matching size 7;
- B8 refresh plan:
  - 19 calls,
  - 532 pair-object slots,
  - median refresh time 22.61 s;
- canonical end-to-end result at `m = 20`:
  - 26.84 s versus 120.57 s for full refresh,
  - 4.49× speedup,
  - repair rate 1.0;
- action-aware multi-cell result:
  - 90 retire actions,
  - 51 refresh actions,
  - 15 calls,
  - 413 slots,
  - 1.41× speedup over B8,
  - all 1,307 audited supported cells repaired to Q-error at most 2.

## Canonical timing protocol

The paper reports the canonical Phase-5F protocol.

For the main `m = 20` experiment, the reported end-to-end time is:

```text
screening/scoring + exact PostgreSQL verification + measured refresh
```

The canonical median decomposition is approximately:

```text
0.066 s + 3.12 s + 23.64 s = 26.84 s
```

The full-refresh baseline is 120.57 s, yielding a 4.49× speedup.

### Exploratory clean-accounting variant

A separate exploratory script, `run_phase5f_d109_clean.py`, additionally charges pair-count materialization to the end-to-end budget. Its outputs are not used for paper claims.

The exploratory variant is retained only as an accounting sensitivity check:

```text
count materialization + scoring + verification + refresh
```

Do not mix its wall-clock numbers with the canonical paper numbers. Runtime values from different runs may also differ because of machine load, PostgreSQL cache state, and concurrent processes.

## Requirements

### Quick verification

- Python 3.10 or newer;
- packages listed in the supplied requirements file;
- no database server required.

Install dependencies:

```bash
python -m pip install -r requirements.txt
```

If the release contains phase-specific requirement files, install the relevant file instead, for example:

```bash
python -m pip install -r code/phase5_validation/requirements_phase5f.txt
```

### Full PostgreSQL reproduction

The full experiments require:

- PostgreSQL 17 or 18;
- permission to create and drop schemas, tables, and extended statistics;
- Python dependencies listed in the artifact;
- sufficient disk space for ACS snapshots and frozen outputs;
- the reference and current ACS PUMS inputs, or the frozen encoded arrays included with the artifact.

The experiments were developed and validated on Windows using PowerShell. Python runners are platform-independent where they do not invoke Windows-specific wrappers.

## Data provenance

The real-data experiments use official 2019 and 2021 American Community Survey 1-year PUMS person and housing files.

The joined experiment uses:

- 50 selected person attributes;
- 59 selected housing attributes;
- join key `SERIALNO`;
- 109 total attributes;
- 5,886 candidate pairs.

The artifact records the exact sample, deterministic categorical encoding, selected variables, and frozen derived arrays used for the paper.

Raw ACS files are not required for quick verification. Full raw-data reconstruction may download or read the official Census files, depending on the release configuration.

## Full reproduction

Before running database-dependent experiments, configure the PostgreSQL connection string.

Example:

```text
postgresql://postgres:YOUR_PASSWORD@localhost:5432/postgres
```

### Main d = 109 support and runtime experiment

```bash
python code/phase5_validation/run_phase5f_d109.py \
  --dsn "postgresql://postgres:YOUR_PASSWORD@localhost:5432/postgres"
```

The runner reconstructs the stale PostgreSQL state:

1. load the 2019 reference snapshot;
2. collect multivariate statistics;
3. replace rows with the 2021 current snapshot;
4. leave the catalog stale;
5. screen candidate pair-cells;
6. verify selected predicates with PostgreSQL;
7. construct the severe-support graph;
8. execute candidate refresh plans;
9. validate repaired predicates.

### Query-feedback comparison

```bash
python code/phase5_validation/run_phase5g_query_feedback.py \
  --dsn "postgresql://postgres:YOUR_PASSWORD@localhost:5432/postgres"
```

### Frontier audit

```bash
python code/phase5_validation/run_phase5c_frontier_audit.py \
  --dsn "postgresql://postgres:YOUR_PASSWORD@localhost:5432/postgres"
```

Additional experiment-specific arguments and PowerShell wrappers are documented in `code/phase5_validation/README.md`.

## Important definitions

A pair is operationally stale at support floor `m` when at least one current equality cell:

- has current count at least `m`; and
- has stale PostgreSQL Q-error at least 10.

The canonical severe graph at `m = 20` is defined from the floor-aware output:

```text
floor == 20 and pg_severe == 1
```

This yields 141 unique severe pairs. It must not be reconstructed by filtering a single global worst-cell table by support, because the selected worst supported cell can change with the support floor.

## Main output files

Important frozen files include:

```text
results/phase4g_ambient_width/qerrors_d109.csv
results/phase5f_d109_support_runtime/support_pairs.csv
results/phase5f_d109_support_runtime/support_summary.csv
results/phase5f_d109_support_runtime/runtime_trials.csv
results/phase5f_d109_support_runtime/runtime_summary.csv
results/phase5f_d109_support_runtime/end2end_summary.csv
results/phase5g_query_feedback/feedback_summary.csv
```

Later reviewer-validation releases may additionally contain:

```text
results/phase5j_retire_refresh_v2/hybrid_actions.csv
results/phase5k_retire_refresh_classifier/classifier_detail.csv
results/phase6a_nohub_geometry/
results/phase6c_multicell_gate/
results/phase6d_action_aware_multicell_exact/
results/phase6e_verification_cost_bridge/
results/phase6f_theorem_correct_bridge/
```

## Reproducibility notes

- Runtime results are medians over ten randomized repetitions unless stated otherwise.
- PostgreSQL wall-clock results are sensitive to system load, cache state, storage, and server configuration.
- Exact combinatorial counts, graph summaries, classifier decisions, and claim provenance should reproduce exactly from the frozen outputs.
- Small wall-clock variation is expected; structural claims and repair coverage should remain unchanged.
- The retirement guarantee is scoped to the audited supported equality cells. It is not a universal guarantee for every possible future predicate.

## Integrity and provenance

`MANIFEST.csv` records release files and SHA-256 checksums.

`CLAIMS_PROVENANCE.csv` maps paper claims to:

- source file;
- row selector;
- expected value.

Run:

```bash
python validate_artifact.py
```

before using or modifying the release.

## Citation

```bibtex
@article{wang2027evostats,
  author  = {Bilong Wang and Zewen Lyu and Xu Song},
  title   = {EvoStats: Geometry-Aware Maintenance of Multivariate Statistics under Dependency Drift},
  journal = {Proceedings of the VLDB Endowment},
  year    = {2027},
  note    = {Submitted}
}
```

Update the volume, issue, pages, and DOI after publication.

## License

Code: add the license selected by the authors, for example Apache-2.0 or MIT.

Data: ACS PUMS data remain subject to the terms and documentation of the U.S. Census Bureau. This repository does not relicense third-party data.

## Contact

- Bilong Wang — `bwan0112@student.monash.edu`
- Zewen Lyu — `zlyu0022@student.monash.edu`
- Xu Song — `xson0048@student.monash.edu`

For reproduction questions, please open a GitHub issue and include:

- operating system;
- Python version;
- PostgreSQL version;
- command executed;
- complete error log;
- whether the run used frozen arrays or raw ACS reconstruction.
