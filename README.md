# Causal-Machine-Learning-Analysis-For-CAMELS-Catchments
## Overview

This repository contains the final analysis pipeline for investigating the effect of **high antecedent rainfall exposure on next-day high-flow occurrence across multiple CAMELS catchments** using causal machine-learning methods.

The analysis combines:

- Multi-catchment CAMELS hydrological observations
- Antecedent rainfall accumulation
- Pre-exposure hydro-meteorological covariates
- Catchment fixed effects
- Linear Double Machine Learning (DML)
- Causal Forest
- Chronological training and held-out evaluation
- Common-support assessment
- Catchment-level Conditional Average Treatment Effects (CATEs)
- Treatment-effect heterogeneity analysis

The primary analysis is conducted across **50 CAMELS catchments**.

---

## Research Question

The analysis addresses the following question:

> **How does high antecedent rainfall exposure affect the probability of next-day high-flow occurrence, and how does this effect vary across catchments with different hydro-meteorological characteristics?**

The analysis treats antecedent rainfall as an explicit exposure rather than using rainfall only as a predictive variable.

---

## Analytical Framework

The causal structure is:

```text
Antecedent rainfall
        ↓
High rainfall exposure
        ↓
Next-day high-flow occurrence
