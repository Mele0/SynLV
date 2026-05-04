# Reproduce Paper Results

This guide maps analysis-derived SynLV paper tables and figures to executable reproduction targets. It covers outputs that depend on dataset diagnostics, stress-evaluation results, sensitivity analyses, or local real-data summaries. Descriptive scenario, protocol, and implementation tables remain documented in the paper and benchmark docs rather than reproduced as analysis outputs.

Run commands from the repository root.

## Reproducibility Scope

SynLV has three reproducibility layers:

1. Dataset and generator reproduction. Public commands validate the hosted dataset inventory, scenario registry, Croissant metadata, generator entry points, and local split-file schema if the dataset has been downloaded. These checks do not require model checkpoints or result CSVs.
2. Table and figure regeneration from result artifacts. The paper-results CLI rebuilds analysis-derived tables and figures from local long-form synthetic stress results or precomputed summary artifacts supplied through `--results-root` or `--input`. The CLI records input paths and hashes in per-target manifests.
3. Full cold-start benchmark rerun. Training and evaluating every model family across all scenarios, cohort seeds, stress repetitions, and stress-grid cells is supported by the code components but is not a cheap single-command smoke test. See the paper appendix for the reported compute-resource accounting.

The repository does not bundle raw long-form synthetic stress-result CSVs, model checkpoints, model logs, or credentialed MIMIC-IV/eICU-derived row-level files. What is provided here is the dataset/generator validation layer, target registry, aggregation code, expected input schemas, self-tests, and commands for regenerating tables from local result artifacts. See `docs/PAPER_RESULT_ARTIFACTS.md` for the target-by-target input manifest.

## Quick Commands

```bash
python benchmark_analysis/reproduce_paper_results.py --list-targets
python benchmark_analysis/reproduce_paper_results.py --self-test
```

Check whether local inputs are available for a target without generating outputs:

```bash
python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table5_grid_avg --results-root /path/to/local/results
python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table9_primary_da_full --input /path/to/primary_da_full_summary.csv
```

## Synthetic Dataset Validation

Validate the public registry and Croissant inventory:

```bash
python benchmark_release/validate_synlv_release.py --strict 1
```

If local SynLV split files are available, validate their inventory and schema:

```bash
python benchmark_release/validate_synlv_release.py --dataset-root /path/to/SynLV --croissant SynLV_hf_platform_croissant.json
```

## Exact Target Commands

Use `--results-root` for local synthetic model-result outputs, `--dataset-root` for local SynLV split files, and `--input` for precomputed summary artifacts or credentialed local real-data summaries. If several candidate CSV files are found under a root, the CLI stops and asks for an explicit `--input` path.

```bash
python benchmark_analysis/reproduce_paper_results.py --target table3_scenario_mechanism_checks --dataset-root /path/to/SynLV --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target figure2_stress_surfaces --results-root /path/to/local/results --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table5_grid_avg --results-root /path/to/local/results --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table6_severe --results-root /path/to/local/results --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table7_uncertainty --results-root /path/to/local/results --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table8_mice_vs_coxph --results-root /path/to/local/results --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table9_primary_da_full --input /path/to/primary_da_full_summary.csv --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table10_da_alignment_controls --input /path/to/da_alignment_controls_summary.csv --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table_realdata_missingness_near_early --input /path/to/local_realdata_missingness_summary.csv --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table_mimic_da_minus_base --input /path/to/local_mimic_da_minus_base_summary.csv --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table_eicu_da_minus_base --input /path/to/local_eicu_da_minus_base_summary.csv --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table_realdata_reference_conditions --input /path/to/local_mimic_reference_conditions_summary.csv --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table_generator_sensitivity --input /path/to/generator_sensitivity_summary.csv --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table_visit_count_sensitivity --input /path/to/visit_count_sensitivity_summary.csv --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table_stress_suite_results --input /path/to/stress_suite_results_summary.csv --out docs/audit/reproduced_paper_results
python benchmark_analysis/reproduce_paper_results.py --target table_baseline_severe_all --results-root /path/to/local/results --out docs/audit/reproduced_paper_results
```

Targets also accept LaTeX labels, for example:

```bash
python benchmark_analysis/reproduce_paper_results.py --target tab:benchmark_compact_grid_appendix --results-root /path/to/local/results --out docs/audit/reproduced_paper_results
```

## Table and Figure Mapping

| Paper output | LaTeX label | Target ID | Required input | Reproduction mode | Output files | Status |
|---|---|---|---|---|---|---|
| Table 3 scenario mechanism checks | `tab:scenario_mechanism_checks` | `table3_scenario_mechanism_checks` | Local SynLV split files or precomputed scenario diagnostics | Dataset diagnostics from split files or summary wrapping | CSV, TeX, Markdown, manifest | synthetic-public |
| Figure 2 stress surfaces | `figure2_stress_surfaces` | `figure2_stress_surfaces` | Long-form synthetic stress results | Figure regeneration from result artifact | CSV, PNG, PDF, manifest | synthetic-results-required |
| Table 5 Grid avg compact benchmark | `tab:benchmark_compact_grid_appendix` | `table5_grid_avg` | Long-form synthetic stress results | Table regeneration from result artifact | CSV, TeX, Markdown, manifest | synthetic-results-required |
| Table 6 Severe compact benchmark | `tab:benchmark_compact_severe_appendix` | `table6_severe` | Long-form synthetic stress results | Table regeneration from result artifact | CSV, TeX, Markdown, manifest | synthetic-results-required |
| Companion uncertainty table | `tab:benchmark_uncertainty_post_review` | `table7_uncertainty` | Long-form synthetic stress results | Table regeneration from result artifact | CSV, TeX, Markdown, manifest | synthetic-results-required |
| MICE-CoxPH versus CoxPH | `tab:mice_vs_coxph_refresh_compact` | `table8_mice_vs_coxph` | Long-form synthetic stress results containing CoxPH and MICE-CoxPH | Paired table regeneration from result artifact | CSV, TeX, Markdown, manifest | synthetic-results-required |
| Primary DA-minus-BASE full results | `tab:primary_da_full_appendix` | `table9_primary_da_full` | Precomputed paired DA/BASE inferential summary or local paired result artifact | Summary wrapping unless local inferential results are supplied | CSV, TeX, Markdown, manifest | synthetic-results-required |
| DA alignment controls | `tab:da_alignment_controls_summary_appendix` | `table10_da_alignment_controls` | Precomputed alignment summary or local alignment result artifact | Summary wrapping | CSV, TeX, Markdown, manifest | synthetic-results-required |
| Real-data missingness concentration | `tab:realdata_grounding_missingness_near_early` | `table_realdata_missingness_near_early` | Credentialed local MIMIC/eICU-derived summary | Summary wrapping from local credentialed artifact | CSV, TeX, Markdown, manifest | realdata-local-required |
| MIMIC-IV DA-minus-BASE contrasts | `tab:mimic_results_combined` | `table_mimic_da_minus_base` | Credentialed local MIMIC-IV summary | Summary wrapping from local credentialed artifact | CSV, TeX, Markdown, manifest | realdata-local-required |
| eICU DA-minus-BASE contrasts | `tab:eicu_results_combined` | `table_eicu_da_minus_base` | Credentialed local eICU summary | Summary wrapping from local credentialed artifact | CSV, TeX, Markdown, manifest | realdata-local-required |
| MIMIC-IV reference-pair stress response | `tab:realdata_reference_conditions` | `table_realdata_reference_conditions` | Credentialed local MIMIC-IV reference-pair summary | Summary wrapping from local credentialed artifact | CSV, TeX, Markdown, manifest | realdata-local-required |
| Generator sensitivity | `app:results_generator_sensitivity_analysis_appendix` | `table_generator_sensitivity` | Local/precomputed generator-sensitivity summary | Summary wrapping | CSV, TeX, Markdown, manifest | synthetic-results-required |
| Visit-count sensitivity | `tab:visit_count_sensitivity` | `table_visit_count_sensitivity` | Local/precomputed visit-count sensitivity summary | Summary wrapping | CSV, TeX, Markdown, manifest | synthetic-results-required |
| Paired stress-suite results | `tab:stress_suite_results` | `table_stress_suite_results` | Local/precomputed stress-suite summary | Summary wrapping | CSV, TeX, Markdown, manifest | synthetic-results-required |
| Severe degradation across model families | `tab:baseline_severe_degradation_all` | `table_baseline_severe_all` | Long-form synthetic stress results including MICE-CoxPH | Table regeneration from result artifact | CSV, TeX, Markdown, manifest | synthetic-results-required |

For expected schemas and audit commands, see `docs/PAPER_RESULT_ARTIFACTS.md`.

## Excluded Descriptive Tables

The following labels are intentionally excluded because they are descriptive, definitional, or configuration tables rather than analysis-derived paper results:

- `tab:scenario_overview`: scenario definition table.
- `tab:model_input_handling`: descriptive model-interface table.
- `tab:stress_suite_definitions`: sensitivity-suite definition table.
- `tab:impl_repro_details`: implementation details for the controlled reference pair.
- `tab:eval_repro_details`: evaluation protocol and deterministic seeding scheme.
- `tab:mnar_corr_intercepts`: generator parameter documentation unless separately exported by a manifest utility.
- `tab:mimic_cohort_details`, `tab:mimic_features`, `tab:eicu_cohort_details`, and `tab:eicu_feature_details`: cohort/feature real-data tables requiring credentialed local summaries rather than public synthetic result artifacts.

## Real-Data Boundary

MIMIC-IV and eICU are not redistributed. Real-data targets require local credentialed source access or precomputed local summary outputs. The CLI fails clearly when the required local summaries are absent. Do not commit row-level real-data artifacts, derived row-level ICU files, model checkpoints, logs, or private result payloads.

## Troubleshooting

- Missing result files: pass `--results-root` for long-form synthetic result targets or `--input` for summary-only targets.
- Ambiguous candidate files: if several CSV files under `--results-root` match the required schema, pass the intended file explicitly with `--input`.
- Missing columns: long-form synthetic targets require logical fields for scenario, model, cohort seed, prevalence, severity, IBS, and AUC; common aliases are normalized by the CLI.
- Unavailable real-data targets: provide a credentialed local summary via `--input`; row-level clinical data are not bundled in this repository.
- Summary-only targets: some inferential tables use bootstrap/q-value summaries. When only those summaries are available, the CLI wraps the summary and records that provenance in the manifest rather than inventing confidence intervals.
- Full cold-start reruns: use the training/evaluation scripts and local compute environment appropriate for the full benchmark. This is not expected to run as a quick reviewer smoke test; see the paper appendix for the reported compute-resource accounting.
