# Yelp Reviews and Local Business Activity

Master's thesis — Manos Giannitselis

## Overview
Causal analysis of whether Yelp online reviews affect local US county-level 
business activity (employment, establishments, payroll), using staggered DiD, 
IPW-weighted fixed effects, and IV-DiD.

## Methods
- Callaway & Sant'Anna (2021) staggered DiD, doubly-robust
- IPW weights for continuous treatment (change in log reviews)
- Binary, percentile-threshold (60th/75th/90th), and continuous treatment definitions
- PCA-compressed demographic controls (4 components)
- Sector-level heterogeneity: Food, Health, Retail, Other

## Data Sources
- Yelp Academic Dataset (treatment)
- US Census County Business Patterns 2005–2022 (outcomes)
- IPUMS ACS microdata (demographic controls)
- NYT Covid county-level data (controls)

## Scripts
| File | Description |
|------|-------------|
| `Master_file_part1_fixed.Rmd` | Data loading, merging, PCA, panel construction, DiD setup |
| `Master_file_part2_fixed.Rmd` | IPW weights, FE regressions, aggregate and sector DiD |

## Note
Raw data files are excluded from this repository due to size and licensing. 
Run Part 1 fully before Part 2.
