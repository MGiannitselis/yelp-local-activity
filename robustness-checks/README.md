# Robustness Checks

Part of the [Yelp Reviews & Local Business Activity](../) thesis project.

## What's here

`robustness_checks.Rmd` runs three separate robustness exercises
against the main result in [`did-analysis/`](../did-analysis/), using a
small simulated panel with the same structure as the real data.

## 1. IPW-weighted regression

The main specification uses `est_method = "dr"` (doubly robust) in the
Callaway & Sant'Anna estimator (see [`did-analysis/`](../did-analysis/)).
This check re-estimates the effect using inverse-probability weighting
on its own, as an independent approach to the same underlying concern
the doubly-robust estimator's propensity model is meant to address.

The weight for each county-year compares its actual change in review
activity against two things: the overall average change across the
sample, and the change *predicted by that county's demographics*
(`PC1`–`PC4`). Counties whose review growth closely matched what their
demographics would predict get an ordinary weight; counties that
deviated more from that prediction are down- or up-weighted
accordingly.

The reasoning: if certain kinds of counties — by demographic profile —
were simply more likely to see a review surge in the first place,
that's a form of selection into treatment based on observables, and an
unweighted regression would let that demographic-driven selection
contaminate the estimated effect. Weighting by how well a county's
review growth is explained by its demographics is a way of directly
correcting for that selection, rather than relying solely on the
doubly-robust estimator's built-in propensity model to handle it.

## 2. Sector heterogeneity

The main analysis pools all businesses together at the county level.
This check re-runs the event study at the county × sector level (Food,
Health, Retail, Other) to see whether a *different story* emerges when
the data is split this way — whether an effect that looks uniform in
the pooled analysis is actually concentrated in specific kinds of
businesses, or plays out differently across them. This matters for
interpretation: "Yelp activity affects local business activity" is a
different, more specific claim if the effect is really coming from,
say, Food establishments alone.

## 3. Sensitivity to the treatment threshold

[`treatment-definition/`](../treatment-definition/) explains why
treatment is defined by a percentile threshold on review activity
rather than a fixed cutoff, and why multiple percentiles (60th, 75th,
90th) were tried rather than committing to one. This check re-runs the
main DiD specification at each of those three thresholds and compares
the resulting effect sizes directly — if the estimated effect is wildly
different depending on where exactly the threshold is drawn, that's a
sign the result is more about the choice of cutoff than about the
underlying relationship.

## How to read these results

None of these checks "prove" the main result is correct — no robustness
check can do that. What they can do is speak to a few specific ways the
main result could be misleading: demographic-driven selection into
treatment contaminating the estimate (check 1), a false impression of
uniformity across business types (check 2), or sensitivity to an
arbitrary threshold choice (check 3). Passing all three doesn't
eliminate the deeper identification concerns discussed in
[`did-analysis/`](../did-analysis/) (e.g. whether treatment timing
itself is exogenous) — those aren't the kind of thing a robustness
check can resolve, only a better dataset could.

## Summary

| Check | What it speaks to |
|---|---|
| IPW-weighted regression | Whether demographic-driven selection into treatment is contaminating the estimated effect |
| Sector heterogeneity | Whether the pooled effect looks the same, or tells a different story, split by business type |
| Threshold sensitivity (60th/75th/90th) | Whether the result depends heavily on exactly where the treatment cutoff is drawn |
