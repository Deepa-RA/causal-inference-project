# Causal Inference & Large-Scale Experimentation — Learning Guide

**Purpose:** a portfolio project that demonstrates causal inference and large-scale A/B testing skills, built to speak directly to a Preply Senior Data Scientist (Insights) job description. This guide documents every step so you could rebuild the whole project from nothing, using only this file.

**How this works:** you write and run all the code yourself. Claude's job is to explain the concept before each step, tell you what to build (not hand you the finished code), answer questions when you're stuck, and review what you produce. If you ever get a full code block from Claude in this project going forward without asking for one, that's a sign we've drifted from the plan — call it out.

Each module below has: **what you'll learn**, **what you'll build**, and a **checkpoint** — a question you should be able to answer before moving to the next module. If you can't answer the checkpoint, that's the signal to slow down, not skip ahead.

---

## Module 0 — Environment setup (already done — read this to understand what exists)

**What "the environment" means:** before you can write or run any code, you need three things: a place for the project's files to live, a way to track changes to those files over time, and the software libraries the code will depend on. That's what this module set up.

**What was actually done, and why:**

1. **Project folder created:** `Documents/causal-inference-project` on your Mac. This is the single folder that will hold everything — data, code, notes — for this project.
2. **Git repository initialised** (`git init`). Git tracks every change you make to your files over time, so you can see history, undo mistakes, and eventually push the finished project to GitHub. Command: `git init`, run inside the project folder.
3. **Folder structure created:**
   - `data/raw/` — the simulated data will land here, untouched, once generated.
   - `data/processed/` — any cleaned-up or derived versions of the data go here instead of overwriting the raw files.
   - `notebooks/` — one Jupyter notebook per session/module, numbered in order (e.g. `00_simulate_data.ipynb`, `01_eda.ipynb`, ...).
   - `src/` — for any reusable Python code you pull out of notebooks later (not needed yet).
4. **`.gitignore` file added** — tells git to ignore things that shouldn't be tracked: the actual data files (they can be large and are reproducible from the simulation code), Python cache files, and notebook checkpoint files.
5. **Git identity configured**, so commits are attributed to you: name `Deepa-RA`, email `Deepa-RA@users.noreply.github.com` (GitHub's private "noreply" email format — keeps your real email out of public commit history, same pattern worth using on your other repos).
6. **Python libraries installed** (via `pip install --user`, meaning installed for your user account rather than system-wide): `pandas` and `numpy` (data handling), `scipy` and `statsmodels` (statistics), `scikit-learn` (machine learning, used later for propensity score matching), `matplotlib` (plotting), and `jupyter` (notebooks).
7. **Initial commit made** — the empty scaffold (folders, `.gitignore`, a placeholder `README.md`) was committed to git as a starting point.

One wrinkle you don't need to worry about: there's a leftover `.venv` folder in the project directory that came pre-existing in this workspace and can't be deleted — it's harmless, already excluded via `.gitignore`, and not part of the actual project.

**Checkpoint:** if someone deleted this whole folder tomorrow, could you explain the three things you'd need to set back up, and why each one matters? (A place for files, version tracking, and the right libraries.)

---

## Module 1 — Design & simulate the dataset

**What you'll learn:** how to translate a business scenario into a simulated dataset with a known "ground truth" — meaning you'll build data where you, the creator, know the real effect of the product change, because you're the one deciding what that effect is. This matters because it's the only way to check whether your analysis methods later actually recover the truth, rather than just producing a plausible-looking number.

**The scenario** (confirm or adjust before building):

- A company rolls out a product change **city by city**, and the rollout order isn't random — some cities get it earlier for business reasons (bigger markets, easier logistics). This is what makes a naive before/after comparison misleading: cities that got the change early might already have been different from cities that didn't.
- Inside cities that get the change, there's also a **proper randomised A/B test** run on individual users — a "clean" experiment nested inside the "messy" rollout.
- Neighbouring cities aren't fully independent — some of the effect **spills over** between them. This breaks an assumption most basic A/B testing analysis quietly relies on (that one user's outcome doesn't depend on another user's treatment).

**What you'll build:** a Python script or notebook (`notebooks/00_simulate_data.ipynb`) that generates:
- A city-level panel (one row per city per time period) with a non-random rollout flag, city characteristics that influence rollout timing (this is your "selection bias"), and an outcome metric.
- A user-level file for the nested A/B test, with randomised treatment assignment inside treated cities.
- A rule that lets nearby cities' outcomes be nudged by their neighbours' treatment status (your spillover mechanism).

**Checkpoint:** without looking at your code, can you say in one sentence each: (1) what makes the city rollout biased, (2) what makes the nested test clean, and (3) what spillover means here and which real-world Preply dynamic it stands in for?

---

## Module 2 — EDA & KPI framing

**What you'll learn:** how to choose and justify a primary metric before running any statistical test — a real Senior DS is expected to define KPIs, not just analyze whatever column is available.

**What you'll build:** exploratory analysis confirming the bias and spillover you built are actually visible in the data, plus a documented choice of primary outcome metric (framed as retention/engagement, not a generic "conversion" number) and 1–2 guardrail metrics.

**Checkpoint:** why might a company want to track a guardrail metric alongside its primary metric — what could a primary-metric "win" hide?

---

## Module 3 — Naive comparison (and why it's wrong)

**What you'll learn:** what confounding actually looks like in numbers, not just in theory.

**What you'll build:** a straightforward treated-vs-control comparison on the primary metric, deliberately naive, to show the biased answer you'd get if you stopped here.

**Checkpoint:** what direction is the bias in, and can you explain why, using the specific way you built the rollout to be non-random?

---

## Module 4 — Confounding control

**What you'll learn:** two standard methods for recovering an unbiased effect estimate from non-random data: stratification and propensity score matching.

**What you'll build:** both methods applied to the city-level rollout, compared against each other and against the (known, since you simulated it) true effect.

**Checkpoint:** in your own words, what does a propensity score represent, and why does matching on it help?

---

## Module 5 — Nested RCT analysis

**What you'll learn:** the full workflow of analysing a proper A/B test: calculating statistical power and minimum detectable effect *before* looking at results, checking primary and guardrail metrics, and reporting a confidence interval rather than just a p-value.

**What you'll build:** full analysis of the nested randomised test.

**Checkpoint:** if this test came back "not statistically significant," what are the two very different explanations you'd need to rule out (no real effect, vs. not enough power to detect a real effect)?

---

## Module 6 — Variance reduction (CUPED)

**What you'll learn:** how to use pre-experiment data to shrink the noise in your estimate, so you can detect smaller effects without needing more users or more time — a genuinely senior-level efficiency technique.

**What you'll build:** CUPED applied to the nested test using your simulated pre-period data, compared against the un-adjusted result from Module 5.

**Checkpoint:** why does using *pre-period* data avoid the trap of "controlling for" something that was itself affected by the treatment?

---

## Module 7 — Geospatial spillover / interference

**What you'll learn:** the concept of SUTVA (the assumption that one unit's outcome doesn't depend on another unit's treatment) and what happens to standard A/B testing when it's violated — a real concern in any networked or marketplace setting.

**What you'll build:** quantify the spillover between neighbouring cities, and discuss what it means for trusting a simple average treatment effect in this kind of rollout.

**Checkpoint:** how does this connect to Preply specifically — where might interference show up between users on a two-sided marketplace?

---

## Module 8 — Multiple testing at scale

**What you'll learn:** what happens to false-positive rates when you're effectively running hundreds of small comparisons at once (one per city), and how false-discovery control addresses it.

**What you'll build:** treat the 300-city rollout as many simultaneous local experiments and apply a correction method.

**Checkpoint:** why does running 300 comparisons at a 5% significance threshold not mean a 5% chance of a false positive overall?

---

## Module 9 — Synthesis & README

**What you'll learn:** how to turn an analysis into a business narrative — the actual deliverable a stakeholder reads.

**What you'll build:** cohort/retention curves as a diagnostic, a short KPI tree, a business recommendation, and a "limitations & what I'd do differently at production scale" section (experimentation tooling, sequential monitoring, what you'd want with real infrastructure).

**Checkpoint:** could you present this project's findings to a non-technical stakeholder in under 3 minutes?

---

## Log

*(We'll add a dated entry here each session — what you built, what you got stuck on, what you learned.)*
