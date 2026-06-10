# THESIS.md
### Aegis — The Research Challenge

*The research-phase framing for Aegis: the full hard problem, the one contribution we stake, the question we ask, and the experiments that answer it. This document owns the research framing. `AEGIS.md` remains the project umbrella; `PRODUCT.md` owns the product; `DECISIONS.md` logs the calls. When the research framing changes, change it here first.*

*Status: research design. The contribution axis is chosen. The research question is drafted as a **target pending the existence experiment** — not locked. Methods are deliberately held open.*

*Last updated: June 2026*

---

## 00 How this document relates to the others

| Document | Owns |
|---|---|
| `AEGIS.md` | The umbrella: north star, the dengue domain, the formal problem, the RL engine, the overall picture. |
| `PRODUCT.md` | The product and engineering: the web tool, screens, architecture, data model. |
| `CURRICULUM.md` | The engineering-PoC build plan (the watered-down regime, to July). |
| `GAP.md` | The **domain** novelty: is "dengue + RL for intervention" a real gap? (Yes.) |
| `GAP_METHODS.md` / `research/DSCOPDWIBMDRPOMDP.md` | The **methods** novelty: has anyone characterized inference-vs-memory by observation structure? (No — clean but narrow gap.) |
| `DECISIONS.md` | The decision log. This document is built on Decisions #14–#17. |
| **`THESIS.md` (this)** | **The research challenge: the landscape, the claimed contribution, the question, the experiments.** |

The PoC is the laboratory; the thesis is the analysis of what the laboratory reveals — with one honesty bolted on: **the watered-down PoC cannot reveal failures it was deliberately built to avoid.** The research contribution does not "emerge" from the PoC on its own. It comes from deliberately re-hardening one axis and studying what breaks.

---

## 01 Research philosophy: three questions, kept separate

Three questions are constantly confused. Keeping them apart is most of the discipline of this project.

- **Is it new? (novelty)** — Found indoors, in the literature. Bounded; you can reason your way to the answer. *Like searching the cliffs and forest for a clearing no one has mapped.*
- **Is it real? (existence)** — Only the world can answer, through experiment. *Like planting in the clearing and seeing what grows.*
- **Is it worth having? (significance)** — Also discovered, not declared: effect size, generality, mechanism. You cannot make the effect bigger by being clever; good design only ensures that *if* the effect is real, you can *see* it.

Novelty is necessary, not sufficient. A novel characterization of an effect that does not exist is worth nothing. So the order is fixed: **resolve novelty first** (done — §07), **confirm existence next** (the first experiment — §08), and **design throughout to detect significance** rather than to inflate it (§09). A well-posed question is one where *both* answers — yes and no — teach the field something.

---

## 02 The full hard problem (the research landscape)

Aegis's difficulty is five dimensions of *one decision*: a barangay health officer, Monday morning, deciding where to act against dengue. Each axis is a feature of that moment. The PoC flattened all five to their easy setting to get a system running; the thesis restores one to full difficulty.

| Axis | The officer's reality | The RL difficulty | PoC simplification |
|---|---|---|---|
| **Observability** *(claimed — §03)* | She sees ~25% of cases, weeks late; the true outbreak is hidden. | The agent acts on a distorted shadow of the true state; a policy optimal for the shadow can be wrong for reality. | Full observability (true counts). |
| Delayed reward | Whatever she deploys, she won't know if it worked for 2–4 weeks; tools pay off on different clocks. | Which of several past decisions caused the change? Tangled credit across overlapping horizons. | 26-week horizon, dense per-week reward — short trail, frequent feedback. |
| Budget | ₱50,000 for the season; she cannot overspend, ever. | Standard RL turns cost into a payable penalty; a hard limit is not payable. | Single scalar budget on *spend* (observable, deterministic → satisfiable by masking). |
| Spatial | "Where to fog this week" is the most concrete thing she needs. | The agent has no spatial state; "where" is bolted on by a heuristic. | Single municipality; barangay targeting = rank-by-reported-cases heuristic. |
| Stochasticity / calibration | No two seasons match; the model is fit to thin, underreported data and is wrong in unknown ways. | A policy can overfit one deterministic simulator and fail when reality differs; calibration is ill-posed. | Deterministic dynamics, fixed literature parameters. |
| *Validation (cross-cutting)* | She'll only trust a tool that works on something real and explains itself. | Not an algorithm problem — it's how we *judge* everything else. | Simulator grades its own agent (near-circular). |

---

## 03 The contribution axis: partial observability

We claim **one** axis: partial observability (underreporting). The other axes live in the environment for fidelity but are **not claimed** — present so the result is about the real problem, controlled so the claim stays attributable (Decision #16).

Why observability:
- It is the richest axis — the field's unquestioned convenience assumption, carried over from COVID, where corroborating signals existed; dengue surveillance has essentially one signal with ~75% underreporting.
- It is product-safe: a hardened observability agent is a drop-in behind the same `/recommend` contract, so the thesis does not force a product redesign (unlike spatial).
- It has a sharp, general, plausibly-novel property hiding inside it (§06).

**The claim-one principle.** Both observability and spatial structure can sit in the environment. Only observability is the thing we claim to advance — with its own controlled comparison and ablation. Claiming two would confound attribution and double the experimental burden on a problem where each axis is its own sub-field.

---

## 04 The two contributions

| Contribution | What it is | Why it stands |
|---|---|---|
| **1. An open, RL-ready dengue environment** | A configurable SEIR-SEI POMDP (Gymnasium-compatible) with knobs for observation structure (and spatial/other axes for fidelity). The instrument that does not currently exist for dengue the way OpenMalaria did for malaria. | Built regardless (it is the method's substrate); a standalone Datasets & Benchmarks-shaped artifact; the fallback if the method is null. |
| **2. A characterization of when inference beats memory** | The empirical study tying the inference-vs-memory advantage to observation *structure* (§05–§09). | The scientific core; a method/finding that generalizes beyond dengue. |

They de-risk each other: if the method underwhelms, the environment stands; if the environment is modest, the method stands. One thesis, plausibly two papers (environment first — lower risk; method second — built on it).

---

## 05 The research question (TARGET — pending the existence experiment)

> **Not locked.** This is the question we are building toward. It is finalized only after the existence experiment (§08) confirms the effect is real. Locking it now would pre-commit before the evidence.

**Overarching (the general claim):** In partially observable sequential decision problems, the value of explicit latent-state inference over observation-history memory is governed by the *structure* of the observation corruption — and specifically, inference becomes necessary, not merely helpful, when the corruption is **correlated with the latent state**.

**The hypothesis being tested:** Memory-based policies degrade gracefully under state-independent corruption but fail systematically when corruption is state-correlated — because the corruption biases exactly the high-stakes regimes, where the agent also has the least data to learn the correction. Explicit inference recovers the lost performance.

**Layered:**
1. *General* — the claim above, about a class of POMDPs.
2. *Motivating instance* — dengue underreporting that worsens during outbreaks is a vivid, high-consequence case of state-correlated corruption.
3. *Generality evidence* — the same effect shown in a stripped synthetic POMDP **and** the realistic dengue environment, so the result is about the property, not the disease. *(This synthetic-environment commitment is itself deferred until existence is confirmed — §11.)*

**The testable pieces:** As state-correlation increases (the knob), how does the performance gap to a full-observability ceiling grow for a memory policy vs. an inference policy? Is any inference advantage attributable to genuine state recovery (does the belief track the truth?) rather than added capacity? Where is the crossover — the structure below which memory suffices and above which inference is necessary?

---

## 06 The sharp property: state-correlated observation corruption

The blurriness the agent sees has two separate qualities:

- **Amount** — how much signal is lost (e.g. seeing 25% vs 50% of cases).
- **Structure** — the *pattern* of the loss. Is it the same always (**state-independent**), or does it depend on the hidden truth (**state-correlated**)?

Dengue reporting collapses during surges: the signal is most suppressed exactly when the true outbreak is largest. That is state-correlated corruption, and it is why memory should fail — it sees the smallest relative signal at the worst moment, with the fewest examples of severe outbreaks to learn from — while inference, modeling the coupling ("reporting drops during surges"), can correct for it.

**Careful wording (survives the "LSTMs are universal approximators" rebuttal):** the inference advantage is expected to be *largest and most reliable* under state-correlated corruption, because that is where memory must learn the hardest, most nonlinear mapping from the sparsest high-stakes data — not because memory is theoretically incapable.

The same structure appears wherever a sensor degrades when the signal spikes: ecological surveys (detection drops as abundance changes), fraud detection (evasion grows with attack scale), sensor networks (degrade under load). That is the generality.

---

## 07 Positioning — what makes the claim novel (and narrow)

The methods-novelty audit (`research/DSCOPDWIBMDRPOMDP.md`, summarized in `GAP_METHODS.md`) returned **a clean but narrow gap**. The contribution must be stated narrowly to survive. We distinguish it explicitly from:

| Prior work | What they varied / showed | Why we are different |
|---|---|---|
| Classical belief-state theory (Åström 1965; Smallwood & Sondik 1973; Kaelbling, Littman & Cassandra 1998) | Belief state is a sufficient statistic — inference dominates finite memory *in the idealized, model-known case*. | We are in the *empirical deep-RL* regime where recurrent policies approximate belief states; theory does not transfer, and says nothing about observation-state coupling. |
| DVRL (Igl et al., ICML 2018) | Inference degrades more slowly as observation-noise *magnitude* rises. | We hold magnitude constant and vary *structure*. Our axis is coupling, not level. |
| Active Inference & RL (Malekzadeh & Plataniotis) | Inference vs. recurrent baseline; gap "more pronounced" under noisy obs (fixed "noisy" vs "partial" task variants). | We run a controlled *sweep of structure*, not a contrast of fixed task types. |
| Rethinking State Disentanglement (Cao et al., AAAI 2025) | Vary distractor variability / SNR; world-model baselines only. | We vary state-correlation and compare against a *recurrent memory* baseline. |
| Recurrent Model-Free Strong Baseline (Ni et al., ICML 2022, arXiv 2110.05038) | A tuned recurrent agent beats specialized POMDP methods on 18/21 envs. | This is the **obstacle, not a competitor** — the bar our inference method must clear. |
| MNAR / informative-missingness statistics | Modeling state-dependent missingness helps *iff* missingness is informative; harmless otherwise. | This is **theoretical scaffolding** — our finding is the RL-control analogue of an established statistical result. |

**The defensible novelty claim:** *to our knowledge, the first work to isolate observation-state coupling — rather than noise magnitude or partial-vs-noisy task type — as the property governing the empirical inference-vs-memory gap in deep-RL POMDPs.*

**The MNAR double edge.** The statistical result is plausibility scaffolding *and* a positioning risk ("isn't this just MNAR ported to RL?"). The defense: the port is non-obvious precisely because a strong recurrent policy might already absorb the correction — whether explicit inference still wins is a genuinely open empirical question. One residual search to run before submission: *MNAR / informative missingness + reinforcement learning + recurrent baseline.*

> ⚠ **Verify before relying.** The audit was produced by a single agent reading mostly abstracts. Confirm DVRL, Ni et al., and Malekzadeh & Plataniotis exist and varied what is stated above — this related-work section rests on those characterizations being accurate.

---

## 08 Experimental design (the crux) and the first experiment

### The crux: hold magnitude constant, toggle structure

The single design constraint the whole contribution lives or dies on. Compare state-correlated corruption against state-independent corruption **at matched magnitude** (equal expected distortion / marginal observation entropy). Then any inference advantage can only be attributed to *structure*, not amount.

> **Failure test:** if a reviewer can restate the result as "more noise → inference helps more," the framing has collapsed into DVRL. Matched magnitude is what prevents that.

This constrains exactly *one* comparison — the one that proves the point. It does not restrict the environment, the method, or any other experiment. (It is a scalpel on one test, not handcuffs on the project.)

### The apparatus: three agents

| Agent | Observability | Role |
|---|---|---|
| Full-observability policy | True state | Ceiling — what perfect information buys. |
| Strong recurrent policy | Distorted observations | The memory baseline. **Must be tuned to the Ni et al. (2022) standard** — beating a weak LSTM proves nothing. |
| Inference policy | Distorted observations | The contender (world model / belief method — *candidate, not committed*; Decision #15). |

Measured across the state-correlation knob: how much of the full-obs → partial-obs gap each agent closes.

### The first experiment (the existence / feasibility check)

Before any dengue simulator or world model, the cheapest possible test of whether the effect is even there. A tiny synthetic POMDP (a 1-D latent state with a controllable observation model). Two conditions at matched magnitude: state-correlated corruption vs. state-independent corruption. Two agents: a tuned recurrent baseline and a full-observability ceiling. One plot.

**The gate:** does the memory agent's gap-to-ceiling blow up *specifically* in the state-correlated condition, relative to matched-magnitude independent noise?
- **Yes →** the phenomenon exists. Proceed; lock the RQ (§05) and the generality commitment (§11).
- **No (gap similar in both) →** the effect may not be there. Reframe or fall back to dengue-as-contribution. *(Honest caveat: a quick null may mean an undertuned baseline or too-easy toy, not a true absence — investigate before abandoning.)*

This toy is also the **abstract end of the generality evidence** (§09) — the existence check and the generality arm are the same experiment at two levels of polish. That is why it is worth running first.

---

## 09 The significance plan (designed in, not bolted on)

Significance is discovered, but you design to *detect* it — and to avoid the false negative (concluding the idea is worthless when the experiment was merely too clumsy to see a real effect). Four levers:

1. **Effect size.** Pick conditions where the effect should be *large* if it exists; report the gap and whether it grows *sharply* with the knob. A steep, clean trend beats a coin-flip edge.
2. **Generality.** Show the effect in the synthetic POMDP **and** the dengue environment — two settings sharing only the property. This converts "a dengue finding" into "a finding about a class."
3. **Mechanism.** Explain *why* — inference wins because corruption hides the truth at the worst moment and memory has the least data to correct it. A mechanism is citable; a number is forgettable.
4. **Anchoring.** Frame the result as the RL-control analogue of the established MNAR/informative-missingness result — important rather than arbitrary.

The matched-magnitude design and the strong baseline are *soil preparation*: they ensure a flat result means "the effect isn't there," not "I couldn't see it."

---

## 10 Deferred axes (framed, not claimed)

Named precisely so they are acknowledged future work, not hidden holes — and so reviewers see we chose, rather than missed, them.

- **Spatial** — the second-richest axis; resolves the reactive-"where" contradiction, but it is hierarchical/multi-agent RL (a sub-field) and would change the product. Present in the environment; not claimed.
- **Delayed reward / credit assignment** — high ceiling but the short PoC horizon may make it mild; risk of a null. Future axis.
- **Budget / hard constraint** — weakest as framed (observable deterministic spend is satisfiable by masking); the interesting residue is *timing* of spend, a planning problem. Deferred.
- **Stochasticity / calibration robustness** — a strong *supporting* contribution (converts the ill-posed-calibration liability into a robustness result); natural to pair with observability later.
- **Validation** — intrinsic, cannot be designed away; the credibility multiplier. Even one real health officer's feedback would be worth disproportionate trust. Out of scope for the core claim.

---

## 11 Open — not yet decided

Held out of the settled framing until the existence experiment reports:
- **The final research-question wording (§05).** Target, not locked.
- **The synthetic-POMDP generality commitment.** The extra build is committed only once the effect is confirmed real.
- **The method (world model vs. simpler inference).** Candidate, not chosen (Decision #15); the existence experiment also informs whether the heavier machinery is even needed.

---

## 12 Cross-references

- Decisions this document rests on: **#14** (thesis as research challenge, methods open), **#15** (world model demoted to candidate), **#16** (two contributions; observability claimed), **#17** (novelty passed, narrow framing, matched-magnitude design, go/no-go pending experiment).
- Domain novelty: `GAP.md`. Methods novelty: `GAP_METHODS.md` → full audit in `research/DSCOPDWIBMDRPOMDP.md`.
- Umbrella and formal problem: `AEGIS.md` (§02 dengue, §03 formal structure, §07/§07b RL engine and the world model as candidate method).

---

*End of THESIS.md · This document owns the research framing. When the research question locks, the generality commitment is made, or the method is chosen, update it here and log it in `DECISIONS.md`. The document is more useful with honest "pending" markers than with premature commitments.*