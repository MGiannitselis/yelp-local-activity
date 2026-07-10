# PCA for Demographic Controls

Part of the [Yelp Reviews & Local Business Activity](../) thesis project.

## What's here

`pca_construction.Rmd` — a standalone, runnable version of the PCA step
used in the main analysis. It uses a small simulated dataset with the
same structure as the real panel, so it runs on its own without needing
the rest of the project's data pipeline.

## Why PCA here?

The county-level panel includes seven demographic/economic control
variables: percent employed, average education, total population,
percent female, average age, percent white, and real average income.

Using all seven directly as controls in a regression is problematic for
two reasons:

1. **Correlation.** Several of these variables move together (e.g.
   education and employment, income and education), so including them
   all separately inflates standard errors and makes individual
   coefficients hard to interpret (multicollinearity).
2. **Dimensionality.** Seven controls, each interacted with county and
   year fixed effects, adds a lot of noise relative to the signal
   available in a county-year panel of this size.

PCA re-expresses the seven correlated variables as a smaller number of
*uncorrelated* components, each capturing a distinct axis of variation
in the underlying demographics. Instead of seven noisy, correlated
controls, the regressions in [`did-analysis/`](../did-analysis/) use
four clean, orthogonal ones (`PC1`–`PC4`).

There's also a conceptual motivation beyond fixing multicollinearity:
a county's demographic profile isn't really seven independent facts —
it's a *combination* of characteristics that tend to move together
(e.g. income, education, and employment jointly describing something
like "socioeconomic status"). PCA surfaces those underlying combinations
directly, so the controls used in the regressions reflect how these
characteristics actually co-vary across counties, rather than treating
each variable as if it told an independent story.

## Why standardize first

`prcomp(..., center = TRUE, scale. = TRUE)` puts every variable on the
same scale (mean 0, standard deviation 1) before running PCA. This
matters because population is measured in the tens of thousands while
percentages sit between 0 and 1 — without standardizing, PCA would be
dominated by whichever variable has the largest raw numbers, not the
variables with the most explanatory content.

## Why 4 components, not fewer or more

The scree plot shows how much variance each successive component
explains; the rule of thumb is to keep components up to the "elbow,"
where adding another component stops buying much additional explained
variance. With 7 input variables, 4 components typically capture the
bulk of the variation while leaving a 5th+ component that mostly
reflects noise. Too few components risks discarding real signal (e.g.
dropping an axis that distinguishes county size from everything else);
too many defeats the point of PCA, since near-zero-variance components
just add estimation noise back in.

## Interpreting the components

Raw PCA output gives you components, not meaning — the loadings
(`pca$rotation`) show which original variables each component is mostly
built from. In this project, the four retained components loaded
roughly as:

- **PC1** — general demographic & socioeconomic status (income,
  education, employment moving together)
- **PC2** — employment/education vs. racial composition (an axis where
  these move in opposite directions)
- **PC3** — county size (population dominates this component)
- **PC4** — gender balance vs. racial composition

*(Worth double-checking these labels against your own loadings output
before treating them as final — they were inferred from labels used
later in the project's regression tables, not re-derived from scratch
here.)*

Labeling components like this turns "PC1, PC2, PC3, PC4" in a
regression table into something a reader can actually reason about,
instead of four anonymous numbers.

## How the output gets used downstream

`scores` is a county-year table with four PCA-derived columns
(`PC1`–`PC4`). It gets left-joined onto the main panel and used as a
control in the DiD/IV regressions, e.g.:

```r
panel_with_controls <- panel %>%
  left_join(scores, by = c("county_name", "year"))

feols(emp ~ treatment + PC1 + PC2 + PC3 + PC4 | county_name + year,
      data = panel_with_controls)
```

## Summary

| Step | Purpose |
|---|---|
| Standardize (`center`, `scale.`) | Puts variables measured in different units on equal footing |
| Run `prcomp()` | Finds uncorrelated linear combinations of the inputs |
| Scree plot | Decides how many components carry real signal |
| Inspect loadings | Turns each component into something interpretable |
| Extract scores | Produces clean, orthogonal controls for downstream regressions |
