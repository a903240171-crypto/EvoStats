# EvoStats Reproducibility Artifact

Paper: **EvoStats: Geometry-Aware Audit and Maintenance of Multivariate Statistics under Dependency Drift**  
Authors: **Bilong Wang, Zewen Lv, Xu Song, Ziqi Zhang, Nan Wen, Yichao Zhu**  
Target: PVLDB Volume 20 / VLDB 2027.

This package contains the code, frozen measurements, derived data, and figure
sources used to evaluate EvoStats. It covers the original audit and planning
experiments together with the later query-consequence, execution-scaling,
multi-state, lifecycle, and leave-one-hub-out evaluations.

## Quick Check

Python 3.11 and PostgreSQL 17 were used for the reported runs. The integrity and
claim checks use only the Python standard library.

```powershell
python verify.py
python reproduce_tables.py
```

The second command regenerates six compact paper-facing tables under
`tables/generated/` directly from the stored CSV files.

Install the full dependencies before rebuilding figures or running any experiment target (including offline experiment audits):

```powershell
python -m pip install -r requirements.txt
```

To rebuild all six figures from CSV inputs:

```powershell
python figures/build_figures.py --output-dir figures/reproduced
```

The figure builder verifies the SHA-256 hash of every plotted CSV before
rendering PDF, SVG, 600-dpi PNG, preview, and grayscale versions.

## Stored Results

The package includes the measured outputs needed to inspect every paper-facing
claim without a database rerun. `CLAIMS.csv` maps each claim to exact source
files and rows. The most useful summaries are:

- `results/phase5f_d109_support_runtime/support_pairs.csv`: all 5,886 pairs at
  six support floors.
- `results/phase6d_action_aware_multicell_exact/`: one-cell and multi-cell
  action safety.
- `results/phase7a_identified_final/`: query-consequence measurements.
- `results/phase7_remaining_final/phase7b_maintenance_cost/`: California
  maintenance scaling.
- `results/fl_final_evidence/`: independent Florida validation.
- `results/multidrift_core_final/`: CA, TX, FL, and NY drift results.
- `results/final_closeout/`: utility-filtered prevalence, severity, lifecycle
  cost, and break-even analysis.
- `results/phase7h_fixed_evidence_nohub/`: fixed-support leave-one-hub-out
  ablation.

## Database Reruns

Use a disposable PostgreSQL database. Database-backed runners create and drop
experiment schemas. Credentials are never stored in this package.

```powershell
$env:EVOSTATS_PG_DSN = "postgresql://USER:PASSWORD@HOST:PORT/DATABASE"
python run.py list
python run.py experiment leave-one-hub-out
python run.py experiment leave-one-hub-out --postgres --resume
```

`leave-one-hub-out` runs its offline protocol audit by default. Passing
`--postgres` repeats the multi-cell gate and 39 maintenance trials. Other live
runners and their prerequisites are listed in `docs/EXPERIMENTS.md`.

Raw ACS PUMS archives are not redistributed. For the TX/FL/NY multi-state
replication downloads, URLs, byte sizes, and SHA-256 hashes are recorded in
`results/final_closeout/provenance/acs_12_zip_provenance.csv` (12 archives).
The California encoded arrays required for portable paper-facing checks are
included; the public CA source URLs and reconstruction contract are documented
in `docs/DATA.md`.

## Package Layout

```text
code/          experiment implementations
docs/          experiment, data, and provenance notes
figures/data/  exact CSV inputs used by the figure builder
figures/rendered/ submitted figure files
metadata/      portable environment description
results/       raw and summarized experiment outputs
tables/        original and regenerated tables
```

Machine-specific paths, credentials, caches, virtual environments, temporary
files, and superseded no-hub results are excluded. `manifest.json` and
`checksums.sha256` provide byte-level inventory and integrity checks.
