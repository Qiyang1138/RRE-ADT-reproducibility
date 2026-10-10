# RRE-ADT reproducibility

Repository: [Qiyang1138/RRE-ADT-reproducibility](https://github.com/Qiyang1138/RRE-ADT-reproducibility).

## Notebook

`RRE_ADT_reproducibility.ipynb` contains the primary constant-stress accelerated degradation analysis and the additional diagnostic, design, risk and process-family analyses. 
## Requirements

Use Python 3.12 and install the analysis and notebook packages:

```bash
python -m pip install numpy pandas scipy matplotlib ipython jupyterlab nbconvert
```

The notebook uses NumPy, pandas, SciPy and Matplotlib for numerical analysis and plotting. IPython is used to display result tables. Other imports are from the Python standard library.

## Run the complete analysis

Open a terminal in the directory containing the notebook. Set environment variables before starting Jupyter or a new kernel. For example, in PowerShell:

```powershell
$env:ADT_RUN_MODE = 'paper'
$env:ADT_REVIEWER_MODE = 'paper'
$env:ADT_OUTPUT_DIR = 'results'
$env:ADT_REVIEWER_AUTORUN = '1'
$env:ADT_REVIEWER2_AUTORUN = '1'
$env:ADT_REVIEWER3_AUTORUN = '1'
$env:ADT_REVIEWER4_AUTORUN = '1'
python -m jupyter lab
```

Open `RRE_ADT_reproducibility.ipynb` and execute all cells in order with a fresh Python kernel. Later sections read archives and result files created by earlier sections.

For non-interactive execution, use the same environment settings and run:

```bash
python -m jupyter nbconvert --to notebook --execute RRE_ADT_reproducibility.ipynb --output RRE_ADT_reproducibility_executed.ipynb --ExecutePreprocessor.timeout=-1
```

The full workflow performs repeated simulation, model fitting, design searches and bootstrap calculations. Execution time depends on the machine and on whether compatible caches are already present.

## Run modes and configuration

| Environment variable | Default | Function |
|---|---|---|
| `ADT_RUN_MODE` | `paper` | Primary analysis fidelity; accepts `paper` or `audit`. |
| `ADT_REVIEWER_MODE` | Follows `ADT_RUN_MODE` | Diagnostic, risk-reselection and joint-budget analysis fidelity. |
| `ADT_OUTPUT_DIR` | Automatically named directory | Root directory for exported results. |
| `ADT_REVIEWER_AUTORUN` | `1` | Automatically runs the diagnostic and joint-budget analyses. |
| `ADT_REVIEWER2_AUTORUN` | `1` | Automatically runs the risk and screening audits. |
| `ADT_REVIEWER3_AUTORUN` | `1` | Automatically runs the process-family analyses. |
| `ADT_REVIEWER4_AUTORUN` | `1` | Automatically runs the breakpoint analyses. |

Set an autorun variable to `0` to load that analysis's functions without running its main workflow. Result-display cells still require the corresponding files.

For a short primary-workflow check, set `ADT_RUN_MODE=audit`, choose a separate output directory and execute **sections 1–10 only**. The audit mode reduces the primary Monte Carlo settings. The later risk, process-family and breakpoint analyses have their own replication settings and are not all reduced by this switch. Use `paper` mode for the full primary numerical analysis.

Additional configurable quantities include `ADT_BUDGET` (default 12000), `ADT_TMAX_ALLOW` (40000 h), `ADT_MISSION_YEARS` (15), `ADT_EA_RELATIVE_DEVIATION` (0.10) and `ADT_UNCERTAINTY_SEVERITY` (1.0). The primary configuration fixes `N=60`, `m=12` and three stress groups; the separate joint-budget analysis allows `N=30–80`, `m=5–20` and three to five stress groups.

## Analysis sections

| Sections | Analysis |
|---|---|
| 1–10 | Connector calibration, IG model, scenarios, nominal benchmarks, mean-CVaR search, independent validation and primary figures. |
| 11–17 | Model diagnostics, diagnostic-based weights, risk sensitivity, robust benchmarks, joint-budget search and paired intervals. |
| 18–20 | Observation audit, scenario truths, figure labels and timing wrappers. |
| 21–27 | Alternative risk criteria, discrete-scenario checks, tail accounting, screening audit and independent confirmation. |
| 28–34 | Alternative stochastic processes, parameter sensitivity, positive Gamma weights, seed checks and cost matching. |
| 35–39 | High-temperature breakpoint and scenario-weight sensitivity, reselection and independent confirmation. |

## Data and dependencies between sections

Electrical-connector observations at 65, 85 and 100 °C are embedded in the notebook. Calibration does not require a separate input CSV. The observation audit exports the measurements, interval increments and fitting indicators.

The additional analyses read the primary candidate archive, the earlier risk-reselection outputs and compatible simulation caches. A complete sequential run creates these files. To use an existing result archive, the following variables can override the default paths:

| Variable | Directory contents |
|---|---|
| `ADT_R2_BASELINE_DIR` | Primary candidate and screening archives. |
| `ADT_R2_PREVIOUS_EVIDENCE` | Earlier fixed-resource reselection and diagnostic-weight results. |
| `ADT_R3_BASELINE_DIR` | Primary candidate archive for the process-family checks. |
| `ADT_R3_R2_RESULTS` | Risk/screening audit results and caches used by the process-family analyses. |
| `ADT_R4_R2_RESULTS` | Risk/screening audit results and caches used by the breakpoint analyses. |

Use archives produced with the same model, configuration and simulation definitions. Cache-reading routines check their recorded signatures where implemented.

## Outputs

With `ADT_OUTPUT_DIR=results`, the output layout is:

```text
results/
  Table*.csv
  Fig*.pdf
  Fig*.png
  Fig*.tiff
  RRE_*archive.csv
  independent_validation_*raw.csv
  reproducibility_manifest.json
  reviewer_revision_addendum/
  minor_revision_evidence/
  reviewer2_addendum/
  reviewer3_addendum/
  reviewer4_addendum/
```

Key source-data and supplementary outputs include:

| Location | Files and contents |
|---|---|
| Output root | Configuration, fitted parameters, selected designs, independent validation, bias–variance results, search archives and primary figure exports. |
| `reviewer_revision_addendum/` | `R1_empirical_diagnostic_power.csv`, `R1_null_calibration.csv`, `R1_R3_scenario_weights.csv`, `R1_paired_bootstrap_improvements.csv` and `R4_uncertainty_set_provenance.csv`; additional reselection and joint-budget tables. |
| `minor_revision_evidence/` | `connector_observations_and_increments.csv`, `observed_data_schedule_audit.csv`, `scenario_truths_all_targets.csv` and figure-label source data. |
| `reviewer2_addendum/` | `R2_risk_parameter_reaggregation.csv`, `R2_risk_criterion_selected_plans.csv`, tail-mass tables, screening-audit tables, independent confirmation and paired intervals. |
| `reviewer3_addendum/` | `R3_process_parameter_losses.csv`, process comparisons and truths, positive-Gamma-weight reselection and confirmation, seed and cost checks. |
| `reviewer4_addendum/` | `R4_weight_vectors.csv`, `R4_portfolio.csv`, `R4_breakpoint_reselection.csv`, scenario losses, independent confirmation and comparisons against V. |

Raw simulation caches are stored as `.npz` files with associated JSON records in the relevant analysis directories. Primary figures are exported as PDF, PNG and TIFF. Stable output filenames retain the original analysis prefixes.

## Reproducibility

The master seed is `2026`. Simulation seeds are derived from stage, scenario and replication identifiers. Independent validation uses separate stage keys. The primary `reproducibility_manifest.json` records the configuration, package versions, elapsed time and hashes of exported root-level files. Additional analyses produce their own manifests and cache records.

Keep primary search results, fixed-resource reselection results and expanded-candidate audit results associated with their respective analysis stages. Candidate coverage and simulation settings can differ between stages. Compare designs under a common target, scenario distribution and evaluation setting.

## Citation

Please cite the associated article and this repository when using the code or derived results:

*Optimal design of accelerated degradation tests for robust reliability extrapolation under model misspecification*.

Repository: https://github.com/Qiyang1138/RRE-ADT-reproducibility
