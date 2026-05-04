# Paper Result Artifact Manifest

This manifest describes the local inputs needed to regenerate analysis-derived SynLV paper tables and figures with `benchmark_analysis/reproduce_paper_results.py`.

The code repository does not bundle raw long-form synthetic stress-result CSVs, model checkpoints, model logs, or credentialed MIMIC-IV/eICU-derived row-level files. The hosted Hugging Face dataset provides the synthetic split files. Paper-result reproduction is therefore split into three scopes:

- Dataset and generator checks: public commands validate the scenario registry, Croissant metadata, generator entry points, and local split-file schema if the hosted dataset has been downloaded.
- Table and figure regeneration from result artifacts: the reproduction CLI rebuilds tables and figures from local long-form synthetic result CSVs or precomputed summary artifacts supplied through `--results-root` or `--input`.
- Full cold-start benchmark rerun: rerunning all model training and stress evaluation across baselines, scenarios, cohort seeds, repetitions, and stress-grid cells is computationally expensive. See the paper appendix for the reported compute-resource accounting.

## Input Schemas

### Long-Form Synthetic Stress Results

Targets for Figure 2, Tables 5-8, and the appendix severe model-family summary use normalized long-form stress results. The CLI accepts common aliases, but each input must contain these logical fields:

| Logical field | Accepted aliases in the CLI | Notes |
|---|---|---|
| `scenario` | `scenario`, `scenario_name`, `subset`, `hf_subset`, `regime` | Scenario key, subset name, or display name. |
| `model` | `model`, `model_name`, `method`, `estimator` | Model family name. |
| `cohort_seed` | `cohort_seed`, `cohortseed`, `cs`, `seed_cohort` | Independent cohort seed. |
| `rep` | `rep`, `repetition`, `stress_rep`, `mask_rep`, `repeat` | Optional; defaults to 0 if absent. |
| `p` | `p`, `prevalence`, `p_mask`, `p_mask_test`, `mask_prevalence` | Stress prevalence. |
| `K` | `K`, `k`, `severity`, `n_masked`, `mask_k` | Stress severity. |
| `ibs` | `ibs`, `IBS`, `integrated_brier`, `integrated_brier_score` | Integrated Brier score. |
| `auc` | `auc`, `AUC`, `mean_auc`, `td_auc`, `time_dependent_auc` | Time-dependent AUC summary. |

The clean reference is inferred from rows with `p=0` and preferably `K=0`. Degradation is computed as `D_IBS = IBS(p,K) - IBS(clean)` and `D_AUC = AUC(clean) - AUC(p,K)`.

### Scenario Mechanism Diagnostics

`table3_scenario_mechanism_checks` can be regenerated from local SynLV split files or from a precomputed diagnostics table. For split-file mode, the dataset root must contain the registry scenario folders and `cohortseed_000` through `cohortseed_004`, each with `train`, `val`, and `test` split files in CSV or Parquet format.

### Summary-Only Synthetic Tables

DA inferential, alignment-control, generator-sensitivity, visit-count, and sensitivity-suite targets use precomputed local summary artifacts unless the user has local scripts/results for those analyses. The CLI wraps the supplied summary and records its path and hash in the output manifest; it does not invent confidence intervals or q-values.

### Real-Data Local Tables

Real-data targets require credentialed local MIMIC-IV/eICU-derived summaries. Row-level ICU data and derived row-level clinical files are not redistributed in this repository.

## Target Map

| Paper output | LaTeX label | Target ID | Required input artifact | Expected path or pattern | Reproduction mode | Public status | Audit command |
|---|---|---|---|---|---|---|---|
| Table 3 scenario mechanism checks | `tab:scenario_mechanism_checks` | `table3_scenario_mechanism_checks` | Local SynLV split files or diagnostics summary | `--dataset-root /path/to/SynLV` or `--input diagnostics.csv` | Dataset diagnostics from split files or summary wrapping | synthetic-public | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table3_scenario_mechanism_checks --dataset-root /path/to/SynLV` |
| Figure 2 stress surfaces | `figure2_stress_surfaces` | `figure2_stress_surfaces` | Long-form synthetic stress results | `*stress*.csv`, `*benchmark*.csv`, `*summary*.csv`, or explicit `--input` | Table/figure regeneration from result artifact | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target figure2_stress_surfaces --results-root /path/to/results` |
| Table 5 Grid avg compact benchmark | `tab:benchmark_compact_grid_appendix` | `table5_grid_avg` | Long-form synthetic stress results | Same long-form schema | Table regeneration from result artifact | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table5_grid_avg --results-root /path/to/results` |
| Table 6 Severe compact benchmark | `tab:benchmark_compact_severe_appendix` | `table6_severe` | Long-form synthetic stress results | Same long-form schema | Table regeneration from result artifact | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table6_severe --results-root /path/to/results` |
| Table 7 companion uncertainty | `tab:benchmark_uncertainty_post_review` | `table7_uncertainty` | Long-form synthetic stress results | Same long-form schema | Table regeneration from result artifact | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table7_uncertainty --results-root /path/to/results` |
| Table 8 MICE-CoxPH versus CoxPH | `tab:mice_vs_coxph_refresh_compact` | `table8_mice_vs_coxph` | Long-form synthetic stress results containing CoxPH and MICE-CoxPH | Same long-form schema | Paired table regeneration from result artifact | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table8_mice_vs_coxph --results-root /path/to/results` |
| Table 9 primary DA-minus-BASE full results | `tab:primary_da_full_appendix` | `table9_primary_da_full` | Precomputed paired DA/BASE inferential summary or local paired result artifact | explicit `--input primary_da_full_summary.csv` | Summary wrapping unless local inferential results are supplied | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table9_primary_da_full --input /path/to/summary.csv` |
| Table 10 DA alignment controls | `tab:da_alignment_controls_summary_appendix` | `table10_da_alignment_controls` | Precomputed alignment-control summary | explicit `--input da_alignment_controls_summary.csv` | Summary wrapping | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table10_da_alignment_controls --input /path/to/summary.csv` |
| Real-data missingness concentration | `tab:realdata_grounding_missingness_near_early` | `table_realdata_missingness_near_early` | Credentialed local MIMIC/eICU-derived summary | explicit `--input local_realdata_missingness_summary.csv` | Summary wrapping from local credentialed artifact | realdata-local-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table_realdata_missingness_near_early --input /path/to/summary.csv` |
| MIMIC-IV DA-minus-BASE contrasts | `tab:mimic_results_combined` | `table_mimic_da_minus_base` | Credentialed local MIMIC-IV summary | explicit `--input local_mimic_da_minus_base_summary.csv` | Summary wrapping from local credentialed artifact | realdata-local-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table_mimic_da_minus_base --input /path/to/summary.csv` |
| eICU DA-minus-BASE contrasts | `tab:eicu_results_combined` | `table_eicu_da_minus_base` | Credentialed local eICU summary | explicit `--input local_eicu_da_minus_base_summary.csv` | Summary wrapping from local credentialed artifact | realdata-local-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table_eicu_da_minus_base --input /path/to/summary.csv` |
| MIMIC-IV reference-pair stress response | `tab:realdata_reference_conditions` | `table_realdata_reference_conditions` | Credentialed local MIMIC-IV reference-pair summary | explicit `--input local_mimic_reference_conditions_summary.csv` | Summary wrapping from local credentialed artifact | realdata-local-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table_realdata_reference_conditions --input /path/to/summary.csv` |
| Generator sensitivity | `app:results_generator_sensitivity_analysis_appendix` | `table_generator_sensitivity` | Local/precomputed generator-sensitivity summary | explicit `--input generator_sensitivity_summary.csv` | Summary wrapping | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table_generator_sensitivity --input /path/to/summary.csv` |
| Visit-count sensitivity | `tab:visit_count_sensitivity` | `table_visit_count_sensitivity` | Local/precomputed visit-count sensitivity summary | explicit `--input visit_count_sensitivity_summary.csv` | Summary wrapping | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table_visit_count_sensitivity --input /path/to/summary.csv` |
| Paired sensitivity-suite results | `tab:stress_suite_results` | `table_stress_suite_results` | Local/precomputed stress-suite summary | explicit `--input stress_suite_results_summary.csv` | Summary wrapping | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table_stress_suite_results --input /path/to/summary.csv` |
| Severe degradation across model families | `tab:baseline_severe_degradation_all` | `table_baseline_severe_all` | Long-form synthetic stress results including MICE-CoxPH | Same long-form schema | Table regeneration from result artifact | synthetic-results-required | `python benchmark_analysis/reproduce_paper_results.py --audit-inputs --target table_baseline_severe_all --results-root /path/to/results` |

## Artifact Availability in This Repository

The repository includes executable target definitions, aggregation code, schema normalization, self-tests, release validation commands, and documentation. It does not include the long-form stress-result files or checkpoints needed to recreate paper result values without rerunning experiments. Users who have those artifacts can pass them to the CLI; users who do not have them can still validate the public dataset/generator layer and run the aggregation self-test.
