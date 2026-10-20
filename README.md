# Causal Machine Learning for CAMELS Catchments

## Overview

This project investigates the relationship between antecedent rainfall and next-day high-flow occurrence across CAMELS catchments using causal machine-learning methods. The main analysis is a **multi-catchment study covering 668 catchments**, combining daily meteorological forcing data and streamflow observations.

The analysis estimates the average treatment effect (ATE) of unusually high antecedent rainfall on next-day high-flow occurrence and examines how estimated treatment effects vary among catchments.

The main methods are:

- Linear Double Machine Learning (Linear DML)
- Causal Forest DML
- Propensity-score estimation and common-support diagnostics
- Covariate-balance diagnostics
- Catchment-level treatment-effect estimation
- Rainfall-window and treatment-threshold sensitivity analyses

The notebook also contains earlier single-catchment analysis cells and additional reviewer-focused diagnostics. These should be distinguished from the primary multi-catchment analysis when reporting results.

## Research Question

Does unusually high antecedent rainfall increase the probability of a high-flow event on the following day, and does the estimated effect vary across catchments?

The analysis uses observational data. Its causal interpretation therefore depends on the adequacy of measured confounder adjustment, treatment and outcome definitions, overlap, and model assumptions.

## Data

The analysis uses CAMELS forcing and streamflow data.

The notebook includes steps to:

1. Mount Google Drive when running in Google Colab.
2. Locate the CAMELS archive (`all.zip`).
3. Extract the forcing and streamflow archives.
4. Identify catchments with both forcing and streamflow files.
5. Read daily forcing and streamflow records.
6. Construct dates and merge the datasets by date.
7. Add a catchment identifier and sort observations chronologically.
8. Check for duplicate dates, missing values, and physically implausible negative precipitation or streamflow values.

The main multi-catchment workflow uses `basin_id` to distinguish catchments and preserve catchment-level structure.

### Main variables

| Variable | Description |
|---|---|
| `basin_id` | Catchment identifier |
| `Date` | Daily observation date |
| `prcp` | Daily precipitation |
| `streamflow` | Daily streamflow |
| `Rain_lag1` | Previous-day precipitation |
| `Rain3_lag` | Preceding 3-day accumulated precipitation |
| `Rain7_lag` | Preceding 7-day accumulated precipitation |
| `Rain14_lag` | Preceding 14-day accumulated precipitation |
| `HighFlow` | Indicator for streamflow at or above the catchment-specific Q95 threshold |
| `FutureHighFlow` | Next-day high-flow indicator |
| `HighRainExposure` | Binary antecedent-rainfall treatment indicator |

## Treatment and Outcome

### Treatment: high antecedent rainfall

The main treatment variable is `HighRainExposure`. It represents whether antecedent rainfall meets the selected high-rainfall threshold. The primary specification uses a **7-day antecedent rainfall window and a 75th-percentile threshold**.

The notebook also evaluates alternative rainfall windows and thresholds:

- 3-, 7-, and 14-day antecedent rainfall windows
- P65, P75, and P85 rainfall thresholds

For the multi-catchment sensitivity analyses, the threshold is intended to be learned from training data only. The treatment construction used for any reported result should be checked against the corresponding specification in the notebook.

### Outcome: next-day high-flow occurrence

The outcome variable is `FutureHighFlow`, representing whether the following day's streamflow reaches the high-flow threshold.

High flow is defined using a catchment-specific 95th-percentile streamflow threshold (Q95). The notebook constructs the daily high-flow indicator and shifts it by one day to define the future outcome.

The interpretation is therefore the effect of antecedent rainfall exposure on the probability of next-day high-flow occurrence, rather than on streamflow magnitude directly.

## Covariates

The primary multi-catchment model uses the following six covariates:

- `tmax_pre8`
- `tmin_pre8`
- `vp_pre8`
- `streamflow_pre8`
- `season_sin`
- `season_cos`

These represent temperature, vapor pressure, antecedent streamflow, and seasonal variation.

The notebook also examines additional forcing variables, including snow water equivalent (SWE), in earlier data-preparation and diagnostic cells. However, SWE is **not included in the final six-covariate list used in the primary multi-catchment model**. The balance-diagnostic cells explicitly exclude `swe_pre8` because it was not available as a suitable variable in that dataset. The README and manuscript should not describe the final primary model as adjusting for SWE unless the actual fitted specification is changed and verified.

## Analysis Workflow

### 1. Data preparation

Daily forcing and streamflow records are loaded, dated, merged, sorted by catchment and date, and checked for missing or invalid values. Antecedent rainfall accumulations and lagged streamflow variables are then created.

### 2. Treatment and outcome construction

The treatment indicates high antecedent rainfall under the selected rainfall-window and percentile specification. The outcome identifies high flow on the following day using the catchment-specific Q95 threshold.

### 3. Training and held-out data

The notebook separates the multi-catchment data into training and held-out test datasets using its chronological split procedure. The training data are used for fitting the primary models and relevant preprocessing components; the held-out data are used for evaluating estimated treatment effects.

When reporting exact split dates or observation counts, use the final executed notebook outputs rather than assuming that the earlier single-catchment split applies to the multi-catchment analysis.

### 4. Propensity scores and common support

A propensity-score model estimates the probability of high-rainfall exposure conditional on the selected covariates. The notebook compares treated and control propensity-score distributions and restricts the training data to their common-support interval.

The resulting common-support training dataset is named `train_multi_cs`. This step is intended to reduce reliance on extrapolation into regions where treated and untreated observations have little overlap.

### 5. Covariate balance

The notebook evaluates standardized mean differences (SMDs) for the six primary covariates. It uses an absolute SMD below 0.10 as the balance criterion.

It also contains supplementary propensity-score weighting and overlap-weighting diagnostics. These are separate diagnostic analyses and should be reported with their corresponding weighting method, sample, and output file. Balance should not be claimed solely because weighting was performed; it should be demonstrated by the calculated diagnostics.

### 6. Linear Double Machine Learning

The Linear DML model estimates the treatment effect while adjusting for the selected covariates. To manage memory use across the multi-catchment dataset, the notebook samples a bounded number of observations per catchment for model fitting and uses scaling and chunked prediction procedures where appropriate.

The training and held-out treatment-effect estimates are stored in variables including `multi_ate_train` and `multi_ate_test`.

### 7. Causal Forest DML

The Causal Forest DML model provides a flexible estimate of treatment-effect heterogeneity. Its fitting procedure also uses bounded per-catchment samples to manage computational requirements.

The notebook calculates observation-level conditional average treatment effects (CATEs), then aggregates them to catchment-level summaries. Catchment-level results are used to examine the distribution of estimated effects and their potential relationship with catchment characteristics.

### 8. Uncertainty and sensitivity analyses

The notebook includes model-based inference and supplementary catchment-level bootstrap procedures. It also evaluates alternative antecedent-rainfall windows and treatment thresholds.

These uncertainty procedures do not all quantify the same source of uncertainty. In particular, the supplementary catchment bootstrap is conditional on fitted models and does not include model-refitting uncertainty or fully account for within-catchment temporal dependence. It should not be presented as a complete correction for all dependence in the daily observations.

## Main Results and Interpretation

The final verified multi-catchment results are generated in the notebook's final results cells. The model comparison includes:

- Linear DML ATE in the training data
- Linear DML ATE in the held-out data
- Causal Forest ATE in the training data
- Causal Forest ATE in the held-out data
- Catchment-level CATE summaries
- Treatment-effect heterogeneity statistics

The notebook contains multiple uncertainty summaries, including a model-based inference table and supplementary catchment-bootstrap estimates. These results are not interchangeable. For example, one saved summary reports a held-out Linear DML ATE of approximately 0.0716 and a Causal Forest ATE of approximately 0.0704–0.0707, while the reported confidence intervals differ substantially by inference method.

Consequently, numerical estimates and confidence intervals used in a manuscript should be taken from a clearly identified, final results table. The README does not designate one uncertainty procedure as definitive without that choice being reconciled in the analysis.

ATEs for a binary outcome can be interpreted as changes in predicted high-flow probability on the probability scale. For example, an estimate of 0.0716 corresponds to approximately 7.16 percentage points, subject to the model's estimand and assumptions. It does not mean a 7.16% relative increase.

Catchment-level CATEs describe model-estimated variation in the treatment effect. They should be interpreted as estimates with uncertainty, not as proof that catchment characteristics cause the observed heterogeneity.

## Figures and Saved Outputs

The notebook creates and saves analysis results, diagnostic tables, and figure outputs. The main multi-catchment results are saved under:

`/content/multi_basin_causal_results`

When Google Drive is available, the project also uses:

`/content/drive/MyDrive/CAMELS_Causal_Hydrology_Final`

The Drive export workflow includes folders for single-basin results, multi-basin final results, and publication figures.

**Not all manuscript figures are necessarily generated directly by the main notebook.** Some figures may be produced in separate notebooks using saved model outputs, exported tables, or catchment-level estimates. Each manuscript figure should therefore be mapped to its actual source notebook or script and the corresponding saved data file.

Before submission, verify that:

- Each figure uses the intended 668-catchment analysis or clearly identifies a different sample.
- Model-comparison figures use estimates from the selected final results table.
- Catchment heterogeneity plots use the final held-out CATE results and catchment-level aggregation.
- Covariate-balance figures identify whether they show unweighted, propensity-score-weighted, or overlap-weighted diagnostics.
- Robustness figures identify the rainfall window and percentile threshold used.
- Figures generated in separate notebooks are reproducible from the saved outputs referenced by those notebooks.

Older single-catchment figures should not be presented as results from the multi-catchment analysis unless they are explicitly labelled as a separate analysis.

## Reproducibility

The notebook is designed primarily for Google Colab and uses Google Drive for input data and result storage.

To reproduce the workflow:

1. Place the CAMELS archive at the path expected by the notebook, or update the input path.
2. Run the data-extraction and preparation cells.
3. Run the multi-catchment treatment and outcome construction cells.
4. Run the training/held-out split and covariate-preparation cells.
5. Fit the primary Linear DML and Causal Forest DML models.
6. Run common-support and balance diagnostics.
7. Generate held-out treatment effects and catchment-level summaries.
8. Run the chosen uncertainty and sensitivity analyses.
9. Export the final tables and figures.
10. Confirm that the manuscript uses the same model outputs and sample definitions.

The notebook uses reproducible random seeds in several sampling and bootstrap steps. Because some model-fitting procedures use bounded samples, reproduce the corresponding sampling settings when rerunning or comparing results.

## Limitations

- The analysis uses observational data, so causal interpretation depends on the assumptions required for identification.
- Propensity-score overlap and covariate balance diagnostics address only measured covariates and do not rule out unmeasured confounding.
- Daily hydrological observations can be temporally dependent.
- Catchment-level bootstrap results conditional on fitted models do not include all sources of model uncertainty.
- Bounded per-catchment sampling reduces computational demands but means some fitting stages do not use every available observation.
- Estimates of treatment-effect heterogeneity depend on model specification, sample coverage, and the reliability of catchment-level aggregation.
- Different uncertainty procedures in the notebook can produce different confidence intervals; the manuscript should identify the chosen inference method and explain its scope.

## Project Structure

The notebook's output organization is intended to separate:

- **Single_Basin_Results** — results from the earlier single-catchment workflow
- **Multi_Basin_Final_Results** — primary multi-catchment model outputs and diagnostic tables
- **Publication_Figures** — figures prepared for reporting or publication

Exact filenames should be taken from the exported directory listing, because output names may differ across notebook revisions.

## Final Reporting Checklist

Before using results from this project in a manuscript:

- Confirm that the result comes from the 668-catchment multi-basin analysis when that is the intended study.
- Confirm the treatment threshold, rainfall window, outcome definition, and covariate set for the selected model.
- Identify whether each estimate is training-based or held-out.
- Report the inference method alongside each confidence interval and p-value.
- Keep unweighted balance results separate from weighted balance results.
- Label supplementary robustness analyses separately from the primary specification.
- Ensure all manuscript figures can be traced to the correct notebook, script, and saved output.
