# Staggered Difference-in-Differences (Callaway & Sant'Anna)

Part of the [Yelp Reviews & Local Business Activity](../) thesis project.

## What's here

`did_analysis.Rmd` — a standalone, runnable version of the core DiD
estimation, using the `did` package's `att_gt()`/`aggte()` functions. It
runs on a small simulated panel with the same structure as the real
data (see [`treatment-definition/`](../treatment-definition/) for how
`g`, the first-treated year, is actually constructed, and
[`pca-explained/`](../pca-explained/) for `PC1`–`PC4`).

## Why staggered DiD, not a standard two-way fixed effects regression

Counties don't all cross the "high review activity" threshold in the
same year — some are treated in 2018, others not until 2021, and some
never are. A standard two-way fixed effects (TWFE) regression with a
single treatment dummy is known to produce biased estimates under this
kind of staggered adoption, because it implicitly uses already-treated
counties as part of the comparison group for later-treated ones. The
Callaway & Sant'Anna (2021) estimator avoids this by estimating the
treatment effect separately for each treatment-cohort/time-period
combination, then aggregating those up cleanly.

## Key design choices

**`control_group = "notyettreated"`** — at each point in time, the
comparison group is counties that haven't been treated *yet* (rather
than only counties that are never treated). This was chosen partly
because the never-treated group was fairly small given the overall
sample size, and partly because it tells a more interesting empirical
story — using not-yet-treated counties as controls captures more of
the variation in the data than restricting to a small, fixed
comparison group.

**`est_method = "dr"` (doubly robust)** — combines a regression-based
outcome model with a propensity-score model for treatment timing. This
was largely the package's default estimation method rather than a
deliberate choice argued for over the alternatives (outcome-regression-only
or propensity-score-only). A separate IPW-based approach was run later
as a robustness check (see [`robustness-checks/`](../robustness-checks/)),
which is a more direct way of probing sensitivity to this choice than
arguing for `dr` on first-principles grounds here.

**`xformla = ~ PC1 + PC2 + PC3 + PC4 + region_name`** — the same four
PCA-derived demographic controls used throughout the project (see
[`pca-explained/`](../pca-explained/)), plus region fixed effects, so
group-time effects are estimated net of demographic composition and
regional differences.

**`bstrap = TRUE, biters = 200, cband = TRUE`** — standard errors and
confidence bands come from a multiplier bootstrap (200 iterations)
rather than closed-form formulas, which is more reliable with a panel
of this size and gives simultaneous confidence bands valid across the
whole event-study path, not just pointwise.

## From group-time effects to one result

`att_gt()` produces a separate estimate for every treatment-cohort ×
calendar-year combination — informative, but not a single headline
number. Two aggregations are used:

- **`type = "simple"`** — one overall average treatment effect on the
  treated, collapsing across cohorts and time.
- **`type = "dynamic"`** — effects organized by *event time* (years
  since treatment, not calendar year), which is what the event-study
  plot shows. This is the more informative version, since it reveals
  how the effect evolves before and after treatment, not just its
  average size.

## Reading the event-study plot

Two things to look for:

1. **Pre-treatment estimates (event time < 0) close to zero** — this is
   the empirical check for the parallel trends assumption the whole
   design relies on. If treated and not-yet-treated counties were
   already diverging before treatment, the design's core assumption is
   in trouble.
2. **Post-treatment estimates (event time ≥ 0)** — the actual effect of
   crossing the review-activity threshold, and whether it grows,
   shrinks, or stays flat as time since treatment passes.

## What this doesn't solve, and why C&S was still the right call to lead with

Before settling on Callaway & Sant'Anna as the main specification, a
range of other approaches were tried and compared — standard TWFE, an
IV-based specification, and other staggered-DiD variants — partly to
understand how sensitive the results were to the choice of estimator,
and partly because working through several methods surfaces a better
understanding of what each one actually assumes.

C&S was chosen to lead with because it directly addresses the
staggered-adoption bias that plain TWFE is known to have, and because
the never-treated group in this sample was small enough that using
not-yet-treated counties as controls made better use of the data (see
above). But it doesn't resolve two deeper identification concerns that
remain open questions rather than solved problems:

- **Possible reverse causality in treatment timing.** If a business
  actively encourages customers to leave reviews once it's already
  doing well, "crossing the review-activity threshold" could partly be
  a *symptom* of local business success rather than its cause — i.e.
  treatment timing itself may not be as good as random.
- **Uneven baseline access to the platform.** Not every county had
  equal access to, or awareness of, Yelp from the start of the sample,
  which complicates treating "first crossing the threshold" as a clean,
  comparable event across counties.

Neither issue has an estimator that fully resolves it — a fuller
dataset (e.g. with information on *why* review activity increased in a
given county-year) would help more than switching estimators would. C&S
handles the piece of the identification problem it's designed for
(staggered timing); the rest is a genuine limitation of what's
identifiable from this data, not something a different DiD variant
would fix.

## Summary

| Choice | Reason |
|---|---|
| Callaway & Sant'Anna over TWFE | Staggered adoption timing biases standard TWFE |
| `notyettreated` control group | Never-treated group was small; tells a more interesting story than a narrow fixed comparison |
| Doubly robust estimation | Package default; sensitivity checked later via a separate IPW approach |
| PCA controls + region FE | Nets out demographic composition and regional differences |
| Bootstrap, simultaneous bands | More reliable inference at this panel size |
| Dynamic (event-study) aggregation | Shows the full time path, and lets pre-trends be checked directly |
