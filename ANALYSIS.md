# Analysis & Methodology

This document walks through the reasoning behind each design and analysis decision in this project, module by module, recorded as the project was built rather than reconstructed afterwards. See [README.md](./README.md) for the project overview and how to reproduce it.

---

## Module 0 — Environment Setup

Standard project scaffolding: a dedicated project folder, a git repository for version history, and the Python libraries needed for data handling (`pandas`, `numpy`), statistics (`scipy`, `statsmodels`), machine learning (`scikit-learn`, used later for propensity score matching), plotting (`matplotlib`), and notebooks (`jupyter`).

---

## Module 1 — Design & Simulate the Dataset *(in progress)*

### The scenario, and why it's shaped this way

The "treatment" is a **geo-targeted ad campaign**, not a change to the product itself — this matters because the product is available to everyone on the platform regardless of location, so geography can only plausibly gate *exposure* through something like advertising, not the product itself.

Cities are selected for the campaign based on **growth potential** — a stand-in for the kind of business reasoning (market size, momentum) that drives real rollout decisions. Because this same growth-potential score also predicts a city's outcome independent of any campaign effect, comparing treated vs untreated cities without adjustment would be **biased upward**: treated cities would look better partly because they were already trending that way, not purely because of the campaign.

Inside treated cities, a **properly randomised A/B test** is run on individual users. Randomisation means both arms are similar on everything except treatment, so this piece of the design is "clean" — any difference in outcomes between arms is attributable to the treatment itself, unlike the city-level comparison.

### Why staggered, non-random timing

Staggering rollout across time (rather than a single cutoff date) is realistic — real rollouts are usually staggered for operational reasons (capacity, risk management) — and it's also statistically useful, since variation in *timing* (not just treatment status) enables event-study style analysis later. The gold-standard alternative would be a **stepped-wedge design** — staggered, but with randomised order — which would remove the selection bias entirely. This project deliberately uses the messier, non-randomised version instead, because it's what real business-driven rollouts typically look like, and because it's the version that requires the causal inference toolkit this project is built to demonstrate.

### Spillover mechanism

Ad targeting isn't perfectly precise — some campaign reach leaks into neighbouring cities (e.g. shared regional media, people passing through the targeted city). This is modelled as a direct bump to a neighbouring city's outcome, scaled by:
- **Real distance** between cities (closer neighbours get more spillover), using actual city coordinates rather than synthetic positions.
- **Diminishing returns** across multiple treated neighbours, rather than a hard cap — chosen because it mirrors how advertising reach has diminishing marginal value in reality, and avoids the arbitrariness of picking a hard cutoff (e.g. "3 neighbours max") with no principled justification.

### Timeline and seasonality

A 15-month monthly panel (Jan 2025–Mar 2026): 6 months of stable pre-period (needed later for CUPED, which relies on pre-experiment data), a staggered rollout window, and at least 3 months of post-period for every city, including the last to go live.

A **shared seasonal component** is baked into every city's outcome regardless of treatment status, reflecting genuine academic-calendar seasonality relevant to an education marketplace. Rollout timing is kept **independent of season** — deliberately, so the project's central bias remains the single growth-potential confound that Modules 3–4 are built around, rather than stacking a second, entangled calendar-time confound on top of it.

### Effect size

Baseline retention is set around 20–30%, with a true campaign effect of **+3 to +5 percentage points**, chosen to sit in a specific range: large enough that a properly powered analysis can detect it, but small enough that it isn't obvious from a glance at the raw numbers — meaning the naive comparison and the corrected methods should produce visibly different answers, which is the entire point of the later modules.

### Geographic scope

Cities are filtered to **Europe and North America** (45 countries) before sampling, to keep the scenario coherent with a platform whose core markets sit in those regions, rather than diluting the story with a fully global, undifferentiated sample.

### City sampling

300 cities were randomly sampled from ~11,600 candidates in the filtered set, using a fixed random seed for reproducibility.

---

## Module 2 — EDA & KPI Framing
*Coming soon.*

## Module 3 — Naive Comparison
*Coming soon.*

## Module 4 — Confounding Control
*Coming soon.*

## Module 5 — Nested RCT Analysis
*Coming soon.*

## Module 6 — Variance Reduction (CUPED)
*Coming soon.*

## Module 7 — Geospatial Spillover / Interference
*Coming soon.*

## Module 8 — Multiple Testing at Scale
*Coming soon.*

## Module 9 — Synthesis & Business Narrative
*Coming soon.*
