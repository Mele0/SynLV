# Reproducibility Guide

Run commands from the repository root unless stated otherwise.

## Inspect the release scenario registry

```bash
python benchmark_generation/final_generation.py --list-scenarios
python benchmark_release/export_scenario_registry.py --markdown docs/SCENARIO_REGISTRY.md --json docs/scenario_registry.json
```

## Dry-run generation

```bash
python benchmark_generation/final_generation.py --scenario reference --cohort-seed 0 --dry-run
python benchmark_generation/final_generation.py --all-primary --dry-run
```

For a small local smoke cohort:

```bash
python benchmark_generation/final_generation.py --scenario missing_not_at_random_corr --cohort-seed 0 --out /path/to/synlv_smoke --n-patients 100
```

## Validate release metadata

Registry and Croissant-only validation:

```bash
python benchmark_release/validate_synlv_release.py --strict 1
```

Write an explicit report:

```bash
python benchmark_release/validate_synlv_release.py --croissant SynLV_hf_platform_croissant.json --no-download --out docs/audit/release_validation_report.md
```

Local dataset validation, if split files are available:

```bash
python benchmark_release/validate_synlv_release.py --dataset-root /path/to/local/SynLV/root --croissant SynLV_hf_platform_croissant.json --out docs/audit/release_validation_report.md
```

## Compile and inspect entry points

```bash
python -m compileall benchmark_generation benchmark_analysis benchmark_release lib
python benchmark_release/validate_synlv_release.py --help
python benchmark_release/summarize_synlv_release.py --help
```

## Reproducing paper tables and figures

The paper-results reproduction CLI lists analysis-derived table and figure targets and runs a small deterministic self-test of the aggregation logic. The full runbook is `docs/REPRODUCE_PAPER_RESULTS.md`; target input requirements and schemas are summarized in `docs/PAPER_RESULT_ARTIFACTS.md`.

Paper table/figure regeneration is separate from a full cold-start rerun. The CLI consumes local long-form synthetic stress results, precomputed synthetic summaries, or credentialed local real-data summaries supplied with `--results-root` or `--input`. This repository does not bundle raw long-form synthetic stress-result CSVs, model checkpoints, or row-level clinical data. Full cold-start regeneration of every baseline across all scenarios, cohort seeds, and stress-grid cells is computationally expensive and intended for HPC/GPU environments. The repository therefore separates dataset/generator validation and table-level reproduction from documented result artifacts from full benchmark reruns. See the paper appendix for the reported compute-resource accounting.

```bash
python benchmark_analysis/reproduce_paper_results.py --list-targets
python benchmark_analysis/reproduce_paper_results.py --self-test
python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table5_grid_avg --results-root /path/to/local/results
```

## Croissant metadata

The public Croissant metadata file is:

```text
SynLV_hf_platform_croissant.json
```

It describes the synthetic SynLV artifact only. Clinical grounding data are not redistributed.

## Versioning note

These code and documentation updates align the public scenario registry, generation entry points, and validation utilities with the hosted SynLV v1.0 dataset. Dataset rows, hosted parquet files, and Croissant parquet checksums are unchanged.
