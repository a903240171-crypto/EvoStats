# EvoStats

Reproducibility artifact for:

**EvoStats: Geometry-Aware Audit and Maintenance of Multivariate Statistics under Dependency Drift**

Bilong Wang*, Zewen Lv*, Xu Song*, Ziqi Zhang, Nan Wen, Yichao Zhu  
Monash University  
*Equal contribution.

Target: PVLDB Volume 20 / VLDB 2027.

## Artifact

The current frozen submission artifact is:

- `evostats_artifact_submit.zip`
- SHA-256: `4f4b0d25c113719b4b388b2bd37d0d5b4046de74cc3726aea080bcb2351972e7`

It includes the Phase 7 query/lifecycle evidence, the Phase 7H fixed-evidence leave-one-hub-out rerun, the final six figure sources, frozen measurements, provenance metadata, and verification scripts.

After extraction:

```bash
python verify.py
python reproduce_tables.py
```

Expected integrity result:

```text
PASS: all 39 integrity and scientific checks passed.
```

For detailed requirements, database reruns, figure regeneration, and result locations, see `README_EvoStats.md` or the `README.md` inside the ZIP.
