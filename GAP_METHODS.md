# GAP_METHODS.md
### Aegis — Methods-Novelty Gap (companion to GAP.md)

*`GAP.md` answers the **domain** question — is "dengue + RL for intervention" a real gap? (Yes.) This document answers the **methods** question that the thesis's actual contribution rests on — has anyone already characterized when latent-state inference beats observation-memory by the* structure *of observation corruption? This is the clean summary; the full systematic audit lives in `research/DSCOPDWIBMDRPOMDP.md`.*

*Last updated: June 2026*

---

## TL;DR

**Verdict: clean but narrow gap. Moderate-to-high confidence.** No deep-RL paper was found that uses the *structure* of observation corruption — state-correlated/heteroscedastic vs. state-independent/homoscedastic — as the explicit experimental knob governing the empirical advantage of latent-state inference over a strong recurrent memory baseline. The specific causal characterization is unclaimed.

The contribution is **not** pre-empted, but it must be framed narrowly to survive. It is novel *only* as the empirical, deep-RL characterization tying the inference-vs-memory gap to observation-state coupling — not as "inference beats memory" (classical theory) and not as "inference helps under noise" (already shown for noise *magnitude*).

---

## The one nuance that matters

**Classical theory already knows inference dominates memory — in the idealized case.** Belief-state sufficiency (Åström 1965; Smallwood & Sondik 1973; Kaelbling, Littman & Cassandra 1998) settles the model-known theory. It says nothing about the *empirical* deep-RL regime with function approximation (where recurrent policies themselves approximate belief states), and nothing about observation-state coupling as the determining variable. That is the unclaimed space.

---

## What occupies the surrounding territory (and why none pre-empts us)

| Work | What it did | Status |
|---|---|---|
| DVRL — Igl et al., ICML 2018 (arXiv 1806.02426) | Inference degrades more slowly as observation-noise **magnitude** rises (additive Gaussian, swept level). | Near-miss — magnitude, not structure. |
| Active Inference & RL — Malekzadeh & Plataniotis (arXiv 2212.07946) | Inference vs. recurrent baseline; gap "more pronounced" under noisy obs (fixed "noisy" vs "partial" task variants). | Near-miss (closest) — task type, not a structure sweep. |
| Rethinking State Disentanglement — Cao et al., AAAI 2025 (arXiv 2408.13498) | Vary distractor variability / SNR; world-model baselines only. | Adjacent — no recurrent baseline; wrong axis. |
| Recurrent Model-Free Strong Baseline — Ni et al., ICML 2022 (arXiv 2110.05038) | Tuned recurrent agent beats specialized POMDP methods on 18/21 envs. | The **obstacle** — the bar to beat, not a competitor. |
| MNAR / informative-missingness statistics | Modeling state-dependent missingness helps iff informative; harmless otherwise. | Adjacent — same phenomenon, different field. **Scaffolding**, and the largest residual pre-emption risk. |
| ACNO-MDPs — Nam, Fleming & Brunskill, NeurIPS 2021 | RL with a known observe-at-a-cost observation function. | Adjacent — observation function as mechanism, not an inference-vs-memory variable. |
| N-mixture / occupancy models (Royle 2004) | Detection probability covaries with abundance. | Adjacent — estimation, not control; excellent analogy material. |

---

## The defensible novelty claim

> *In deep-RL POMDPs, the empirical advantage of explicit latent-state inference over a strong recurrent memory baseline is governed by the state-correlation structure of the observation process: under state-independent corruption recurrent policies degrade gracefully, whereas under state-correlated corruption (heteroscedastic noise or informative/state-dependent undercounting biasing the highest-state regimes) they fail systematically and explicit inference recovers the lost performance. To our knowledge this is the first work to isolate observation-state coupling — rather than noise magnitude or partial-vs-noisy task type — as the determining property.*

---

## What this means for the design (the binding constraints)

1. **Hold magnitude constant, toggle structure.** The single design that no prior paper has run. Without holding magnitude fixed, the result collapses into DVRL. Failure test: if a reviewer can restate it as "more noise → more help," the framing failed.
2. **Use a tuned recurrent baseline (Ni et al. 2022).** Beating a weak baseline would be dismissed.
3. **Import MNAR as scaffolding, not as a competitor** — frame the finding as the RL-control analogue of the established statistical result.

---

## What would change this verdict

- A deep-RL paper that sweeps observation-noise **structure** (heteroscedastic vs. homoscedastic / informative- vs. uninformative-missingness) and reports the inference-vs-recurrent gap as a function of it → **pre-empts**.
- A paper naming state-*dependence* (not level) as why recurrent policies fail → narrows to a replication.
- **An MNAR-in-RL bridge** porting informative-missingness theory into deep-RL control with a recurrent baseline → the most likely place a pre-empting paper could emerge. **Run this targeted search before submission.**

---

## Caveats

- Negative result by systematic search; *cannot prove a paper does not exist.* All conclusions hold "to our knowledge after systematic search."
- **Terminology-boundary risk is the dominant threat** — the phenomenon travels under many names (heteroscedastic / multiplicative / signal-dependent noise, informative missingness, MNAR, imperfect detection, state-dependent emission). The MNAR-in-RL boundary is especially live.
- ⚠ **Citation verification required.** The audit was produced by a single research agent reading mostly abstracts; only two papers were deeply verified. Before relying on this for related work, confirm DVRL, Ni et al., and Malekzadeh & Plataniotis exist and varied what is stated. Research agents fabricate citations.
- Some surfaced items were 2026-dated preprints (not peer-reviewed) — treat as provisional.

---

*End of GAP_METHODS.md · Full systematic audit: `research/DSCOPDWIBMDRPOMDP.md`. This summary is the canonical methods-novelty reference; the audit is the evidence behind it.*