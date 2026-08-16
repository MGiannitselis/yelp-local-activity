# Yelp Reviews and Local Business Activity

Master's thesis (Speciale) — Manolis Giannitselis

## Overview

Causal analysis of whether Yelp online reviews affect local US
county-level business activity (employment, establishments, payroll),
using staggered difference-in-differences, IPW-weighted fixed effects,
and doubly-robust dose-response methods. County-level panel data
combining Yelp, US Census CBP, and IPUMS ACS demographics.

## How this repo is organized

The full analysis lives in [`main-analysis/`](main-analysis/). The
folders below pull out specific pieces of that analysis — each is
self-contained and runnable on its own, with a README explaining the
reasoning behind the method and the choices made, aimed at readers who
may not be familiar with the technique.

| Folder | What it covers |
|---|---|
| [`main-analysis/`](main-analysis/) | The full thesis code, in two parts: data construction through treatment assignment, then estimation and results |
| [`pca-explained/`](pca-explained/) | Reducing seven correlated county demographic variables into four PCA components |
| [`treatment-definition/`](treatment-definition/) | Defining "treatment" as a Yelp review surge — binary threshold vs. continuous intensity, and why |
| [`did-analysis/`](did-analysis/) | The core method: Callaway & Sant'Anna staggered DiD, event-study estimation, and its limitations |
| [`robustness-checks/`](robustness-checks/) | IPW-weighted regression, sector heterogeneity, and treatment-threshold sensitivity checks |
| [`ipw-dose-response/`](ipw-dose-response/) | Doubly-robust dose-response analysis: does more review growth mean a bigger effect? |

## Methods

- Callaway & Sant'Anna (2021) staggered DiD, doubly robust
- IPW weights for continuous treatment (change in log reviews)
- Binary, percentile-threshold (60th/75th/90th), and continuous treatment definitions
- PCA-compressed demographic controls (4 components)
- Sector-level heterogeneity: Food, Health, Retail, Other

## Data Sources

- Yelp Academic Dataset (treatment)
- US Census County Business Patterns 2005–2022 (outcomes)
- IPUMS ACS microdata (demographic controls)
- NYT COVID county-level data (controls)
