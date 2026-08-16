# Dose-Response: Does More Review Growth Mean a Bigger Effect?

Part of the [Yelp Reviews & Local Business Activity](../) thesis project.

## What's here

`dose_response.Rmd` builds a doubly-robust score for each county-year
and checks whether it varies smoothly with how much review activity
actually grew, rather than just with whether a threshold was crossed.
It runs standalone on a small simulated panel with the same structure
as the real data (see [`robustness-checks/`](../robustness-checks/) for
the same `ipw_wt` construction used here).

## Why this check exists

[`treatment-definition/`](../treatment-definition/) explains why
treatment was ultimately defined two ways — binary (crossing a
percentile threshold) and continuous (raw review intensity) — motivated
by the idea that a real increase in review activity isn't really a
yes/no event; it's a matter of degree. Trying several different
percentile thresholds there was already a step toward that question:
if not a fixed cutoff, then what counts as "enough" growth to matter?

This dose-response check is the natural extension of that same
question, pushed further: rather than picking any single threshold at
all, it looks at whether the estimated effect scales with the *size* of
the increase in review activity, across the full range of review growth
observed in the data.

## How it works

1. **Outcome model** — predicts the change in employment from the
   change in review activity and the PCA demographic controls.
2. **Doubly-robust score** — combines that prediction with an
   IPW-weighted correction, the same doubly-robust logic behind the
   main DiD specification, applied here to continuous review growth
   rather than a binary indicator.
3. **Binning** — counties are grouped into four bins by how much their
   review activity grew, and the average DR score is plotted per bin
   with confidence intervals, to look for a dose-response pattern.

## What this does and doesn't tell you

If the plot shows the average effect rising fairly steadily across
bins, that's consistent with review growth mattering in a genuinely
graded way, not just as an on/off switch — reinforcing that the
continuous treatment framing in `treatment-definition/` was capturing
something real, not just an alternative specification for its own
sake. A flat or non-monotonic pattern would suggest the binary
threshold story is doing most of the work, and the "how much" question
doesn't add much on top of "did it happen at all."

Like the checks in [`robustness-checks/`](../robustness-checks/), this
doesn't resolve the deeper identification concerns raised in
[`did-analysis/`](../did-analysis/) about whether treatment timing
itself is exogenous — it's a check on the *shape* of the relationship,
not on whether the relationship is causal in the first place.
