# Defining Treatment: Yelp Review Surges

Part of the [Yelp Reviews & Local Business Activity](../) thesis project.

## What's here

`treatment_definition.Rmd` shows how "treatment"-> a county experiencing
a meaningful surge in Yelp review activity, is defined, in both binary
and continuous form. It runs standalone on a small simulated panel with
the same structure as the real data (see [`pca-explained/`](../pca-explained/)
for the real panel's construction).

## Why treatment isn't just "Yelp reviews exist"

Yelp review activity grows gradually and unevenly across counties, so
there's no natural on/off switch. The project defines treatment in two
complementary ways, used across different specifications:

#1. Binary: percentile threshold

A county is marked "treated" in the first year its average log-review
count crosses a percentile threshold of that measure across the whole
sample. Everything from that year onward is "post," everything before
is "pre" — this produces a staggered adoption pattern, since counties
cross the threshold in different years (some never do).

**Why a percentile threshold, not a fixed number of reviews:** review
volume grew substantially over the sample period as Yelp itself grew,
so a fixed cutoff (e.g. "50 reviews") would mostly just pick up later
years rather than counties that saw unusually strong review growth
*relative to their peers at the time*. A percentile threshold adjusts
for that.

**Why multiple percentiles (60th, 75th, 90th), rather than one fixed
cutoff:** any single threshold is somewhat arbitrary, and a binary
yes/no treatment doesn't really match the underlying reality — a county
with reviews far above the cutoff and one just barely over it get
treated identically. An early version of this analysis used a single
fixed percentile cutoff; trying several thresholds instead was a way of
probing how the picture changes as the bar for "high review activity"
moves, and was also a stepping stone toward a more natural fix — a
continuous treatment measure — rather than treating any one percentile
as the "correct" cutoff.

### 2. Continuous: review intensity

Collapsing a continuous measure into a 0/1 indicator discards
information — it can't distinguish "just barely crossed the threshold"
from "far exceeded it." The continuous version sidesteps the threshold
question entirely: it uses `avg_log_reviews` directly, interacted with
a post-treatment window, so the estimated effect scales with how much
review activity actually increased. This is the more direct way of
capturing treatment as a matter of degree (dosage), which trying
multiple percentile cutoffs above was already gesturing toward.

Both the binary and continuous versions are carried forward into the
main analysis in [`did-analysis/`](../did-analysis/): the binary
version for the standard staggered DiD design, and the continuous
version for a dose-response-style specification.

## Summary

| Definition | What it captures | Where it's used |
|---|---|---|
| Binary (`did_bin`) | Did a county cross a review-activity threshold? | Staggered DiD |
| Multiple thresholds (60th/75th/90th) | How sensitive is "treated" to where the cutoff is drawn? | Exploring binary treatment before settling on a continuous approach |
| Continuous (`did_cont`) | How much did review activity increase? | Dose-response / intensity models |
