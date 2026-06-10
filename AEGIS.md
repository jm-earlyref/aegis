# AEGIS
### Reinforcement Learning for Vector-Borne Disease Intervention Sequencing

*A shield. A vector. A learning agent.*

---

> *Aegis (n.) — a shield or protection. From the Greek aigis, the protective armor of the gods.*
> *Aedes aegypti — the mosquito that carries dengue.*

---

*Primary umbrella document and living source of truth*
*Last updated: June 2026 — revised following the research-framing pass (Decisions #14–#17) and the methods-novelty audit*

---

## 00 How to use this document

This is `AEGIS.md` — the **umbrella** for the Aegis project. It holds the north star, the dengue domain, the formal problem, the RL engine, and the overall picture. As the project's documentation has grown, specific concerns now live in dedicated files, and this document points to them rather than duplicating them:

| Document | Owns |
|---|---|
| **`AEGIS.md` (this)** | The umbrella: north star, domain, formal problem, RL engine, the whole picture. |
| `THESIS.md` | **The research challenge**: the full hard problem, the claimed contribution, the research question, the experiments. *Read this for anything about the thesis contribution.* |
| `PRODUCT.md` | The product and engineering: web tool, screens, architecture, data model. |
| `CURRICULUM.md` | The engineering-PoC build plan (the watered-down regime, to July). |
| `GAP.md` | The **domain** novelty (dengue + RL). |
| `GAP_METHODS.md` → `research/DSCOPDWIBMDRPOMDP.md` | The **methods** novelty (observation-structure characterization in deep-RL POMDPs). |
| `DECISIONS.md` | The decision log. |

Read **section 01** to re-ground emotionally. Read **02–04** for the technical structure. Read **05** for scope. Read **07/07b** for the RL engine. Read **08** for prior work. Read **THESIS.md** for the research contribution in full. **What is marked ⚠ OPEN** is an explicit undecided design decision; **✓ DECIDED** is resolved and recorded with rationale. Do not re-open decided questions without a specific reason.

> **Note on this revision.** Following the research-framing pass, the thesis now stakes **one** contribution axis (partial observability) rather than "all the hard things at once," and the method for it is **held open** (the world model is the *leading candidate*, not the chosen solution — Decision #15). The detailed research framing has moved to `THESIS.md`; this document now summarizes and points there to avoid drift.

---

## 01 The north star

Almost every Filipino family has a dengue story. In 2024, the Philippines reported 413,960 dengue cases — the highest in ASEAN — with Calabarzon, your home region, among the most affected. This is not an abstract public health statistic. It is the reason this project exists.

The image that grounds this work is concrete: a barangay health officer, on a Monday morning, deciding where to deploy fogging, larviciding, or community source reduction. They have incomplete case reports from last week, a rough rainfall forecast, and ₱50,000 left for the month. Whatever they decide, they will not see the result for two to four weeks. Right now, they make that decision using intuition, local knowledge, and fixed schedules. There is no tool that tells them where to act and when — and actually works.

That is the problem. Not dengue forecasting. Not epidemiological modeling. Not academic benchmarking. The problem is: can we build something that earns that health officer's trust?

> *Can we build an RL agent that learns intervention policies good enough to be trusted by a real health officer, trained on the kind of incomplete, noisy, resource-constrained reality they actually live in?*

The health officer's reality is hard along several axes at once — delayed feedback, incomplete information, a hard budget, spatial choice, a world that won't match the model. That full landscape is the *motivation*. The thesis does not try to conquer all of it; it stakes **one** axis as its scientific contribution and frames the rest honestly as the landscape and as future work. See `THESIS.md` for which axis, why, and what is deferred.

This project is inspired by the AlphaFold model of research: pick a real problem that matters, understand it deeply enough to see what current tools cannot do, and build whatever is necessary. The theory follows the problem, not the other way around. That is why the method for the chosen axis is held open until the problem shows what it demands.

That is what Aegis is.

---

## 02 How dengue actually works

### 2.1 Why dengue is structurally different

Most epidemic modeling — and nearly all epidemic RL — is built for directly transmitted diseases like COVID-19 or influenza, where one infected human passes the pathogen to another human. Dengue does not work this way. It is **vector-borne** (the vector, meaning the carrier, is the *Aedes aegypti* mosquito). The transmission chain is:

1. An infected mosquito bites a susceptible human → the human enters a latent infection period.
2. After roughly five days, the human becomes infectious.
3. Another mosquito bites this infectious human → the mosquito enters a latent infection period.
4. After roughly eight to twelve days, the mosquito becomes infectious — and stays infectious for the rest of its life (mosquitoes do not recover).
5. This infectious mosquito bites more susceptible humans → cycle continues.

This structure has a critical consequence for control: you cannot stop dengue by acting only on humans. You must act on mosquitoes. And the mosquito population is itself driven by rainfall and temperature — not by human behavior. This is why dengue epidemics are strongly seasonal, and why the intervention toolkit is completely different from COVID.

### 2.2 The SEIR-SEI model

**SEIR-SEI** is the standard compartmental model for dengue. It tracks two interacting populations — humans and mosquitoes — each partitioned into disease stages.

#### Human compartments (SEIR)

| Compartment | Meaning |
|---|---|
| **S — Susceptible** | Healthy humans with no immunity. Can be infected by a bite from an infectious mosquito. |
| **E — Exposed** | Infected but not yet infectious. Duration: approximately 5–7 days. |
| **I — Infectious** | Sick and capable of infecting a mosquito. Duration: approximately 5–7 days. |
| **R — Recovered** | Immune to the specific dengue serotype that infected them. |

#### Mosquito compartments (SEI)

| Compartment | Meaning |
|---|---|
| **Sm — Susceptible mosquitoes** | Healthy adult mosquitoes. Population driven by rainfall and temperature. |
| **Em — Exposed mosquitoes** | Carrying the virus, not yet transmitting. Duration: approximately 8–12 days. |
| **Im — Infectious mosquitoes** | Capable of infecting humans. Remain infectious until death. |

#### How the two populations interact

The coupling is through the biting rate. S → E rate depends on Im count. Sm → Em rate depends on I count. This bidirectional coupling means you must manage both populations simultaneously.

> **Scope note (serotypes).** A single SEIR-SEI with one R compartment ignores the four dengue serotypes and antibody-dependent enhancement, under which a second heterologous infection drives severe disease. Defensible for a single 26-week season; flagged honestly as a simplification, since "cases" ≠ "severe cases/deaths," which is what the officer most wants to prevent.

### 2.3 How interventions map to the model

Each intervention targets a different compartment or parameter. This mapping defines the action space.

| Intervention | What it targets | Effect in the model | Effective horizon |
|---|---|---|---|
| **Fogging** | Im — adult mosquitoes | Reduces Im directly. Fast but short-lived (days). | Short — days |
| **Larviciding** | Pre-Sm — larvae | Reduces recruitment into Sm. Effect appears 1–2 weeks later. | Medium — weeks |
| **LSR** | Breeding sites | Reduces carrying capacity. Effect accumulates over weeks. | Long — weeks to months |

**The combo insight:** No single intervention is sufficient. Fogging reduces adult mosquitoes fast but the population rebounds within days if breeding sites are untouched. Larviciding and LSR take weeks to show effect. A rational strategy sequences them — but the right sequence depends on current state, time in season, and remaining budget. Each intervention has a different effective horizon, meaning the agent is dealing with multiple overlapping delayed-reward problems simultaneously. That sequencing problem is precisely what the RL agent must learn.

---

## 03 The formal problem structure

### 3.1 What is a Markov Decision Process (MDP)?

An MDP has five components. Every RL problem is formally an MDP.

| MDP Component | Plain meaning | Dengue mapping |
|---|---|---|
| **State (S)** | Everything the agent observes | Case counts, week, rainfall, budget, recent interventions |
| **Action (A)** | What the agent can do | Deploy fogging / larviciding / LSR in target area |
| **Reward (R)** | Signal after taking action | Cases averted minus intervention cost |
| **Transition (T)** | How world changes | SEIR-SEI ODE simulator |
| **Discount (γ)** | How much to value future rewards | High γ needed — LSR reward arrives weeks later |

**Note on γ:** Discount factor is a real design decision, not a default. A low γ makes the agent myopic — it will bias toward fogging because fogging's reward arrives fastest. A high γ (close to 1) is required for the agent to value the slow-payoff interventions correctly.

### 3.2 The dengue problem as an MDP

#### State

Reported case counts by area for the past N weeks, current week, rainfall index (current and forecast), remaining budget, and recent intervention history.

> **Note on observability:** The state as observed is not the true state. ~75% of dengue cases are unreported, with a 1–2 week lag. The agent never sees true I counts — it sees a noisy, delayed, **state-correlated** fraction of them (reporting worsens during surges). This is the thesis's claimed contribution axis — see §3.4, §4.2, and `THESIS.md`.

#### Action

✓ **DECIDED — Action space: Discrete.** The agent selects from a fixed menu of intervention-area combinations each week. Simpler to train, sufficient for the PoC and baseline research phase. Continuous action space is a v2 upgrade.

#### Reward

The natural reward is cases and deaths averted relative to a no-intervention baseline. Critical challenge: this reward is delayed 3–4 weeks.

> **⚠ OPEN — Reward shaping:** Whether to add intermediate rewards (e.g., reward for reducing larval index) to help learning. Risk: reward hacking. Deferred to research phase after baseline behavior is observed.

#### Transition function

The SEIR-SEI ODE simulator. One call to `step()` advances the simulation by one week.

### 3.3 Constrained MDP (CMDP)

The budget is a hard limit, not a preference. Standard RL folds cost into reward as a penalty — the agent can overspend if it thinks the reward justifies it. A real health officer cannot.

> **⚠ OPEN — Constraint formulation:** Start with Lagrangian PPO for the PoC; compare CPO in the research phase. Watching a naive penalty fail to respect the budget is part of the learning process.

> **Honest flag.** On a single observable scalar *spend* budget, the hard constraint is largely satisfiable by action masking, and the Lagrangian exercise is partly pedagogical. The genuinely interesting residue — *timing* of spend against the seasonal curve — is a planning problem, not a constraint-satisfaction one. Budget is the **weakest** of the candidate research axes; see `THESIS.md` §10 (deferred axes).

### 3.4 Partial observability (POMDP) — the claimed contribution axis

Standard epidemic RL assumes the agent sees true compartment values. This is unrealistic. In the Philippines, ~1 in 4 dengue cases is reported, and reporting **worsens during outbreaks** — the corruption is *correlated with the hidden state*.

✓ **DECIDED — Partial observability is the thesis's claimed contribution axis** (Decision #16). The other hard axes remain in the environment for fidelity but are not claimed.

⚠ **OPEN — the method is held open** (Decision #15). The world model is the *leading candidate*, to be tested against a strong recurrent (memory) baseline. The contribution is **not** the method — it is the *characterization* of when inference beats memory as a function of observation structure. Full framing in `THESIS.md` §03–§09; methods-novelty basis in `GAP_METHODS.md`.

**The three-agent apparatus (research phase):**
1. **Full observability** — the ceiling. What perfect information buys.
2. **Strong recurrent (memory) policy** — the baseline. Must be tuned to the Ni et al. (2022) standard; beating a weak LSTM proves nothing.
3. **Inference policy** — the contender (world model or simpler belief method; *candidate, not committed*).

The finding being hunted: under **state-correlated** corruption (at matched magnitude), the memory baseline fails systematically and inference recovers the lost performance — whereas under matched-magnitude **state-independent** corruption it does not. See `THESIS.md` §08 for the experimental design.

---

## 04 The hard things

### 4.1 Delayed reward and credit assignment

When cases drop three weeks after fogging, the agent can't easily tell which decision caused it — compounded by multiple interventions on different horizons. *Studied as a candidate axis but **deferred** (see `THESIS.md` §10); the short 26-week horizon may make it mild.* Relevant methods: RUDDER, HER, reward redistribution, DreamerV3 — none applied to epidemic control.

### 4.2 Partial observability — *the claimed axis*

~75% of dengue cases unreported; 1–2 week lag; the underreporting is **state-correlated** (worse in surges). The agent flies partially blind, and worse precisely when stakes are highest. ✓ The thesis's claimed contribution (§3.4). Method held open, world model leading candidate. See `THESIS.md`.

### 4.3 Hard budget constraints

A hard limit, not a preference; CMDP formulation in the PoC. The **weakest** candidate axis (see §3.3 flag). Deferred as a contribution.

### 4.4 The combination is the *landscape*, not the *claim*

| Problem | Studied in epidemic RL? | Notes |
|---|---|---|
| Delayed reward | No — all epidemic RL uses dense rewards | Malaria RL used true ODE state, bypassing real delay |
| Partial observability | No — all epidemic RL assumes full observability | A convenience assumption inherited from COVID's multi-signal setting |
| Hard budget constraints | Partially — COVID resource allocation only | |
| Vector-borne SEIR-SEI dynamics | No — all epidemic RL models respiratory disease | |

These four together describe the officer's real problem — the **landscape** that motivates the project. They are **not** the claim. Earlier framing treated "all four together" as the contribution; that has been narrowed (Decision #16). The thesis stakes **one** axis — partial observability — and within it the sharp, plausibly-novel property of **state-correlated observation corruption**. Trying to claim all four at once would confound attribution and is infeasible solo. See `THESIS.md` §03–§04 and `GAP_METHODS.md`.

> **Why narrowing strengthens it.** A narrow, real, precisely-stated gap survives review; a broad "we did everything" claim does not. The contribution is also two-part: the **open dengue environment** (a community artifact) and the **observation-structure characterization** (the method finding). They de-risk each other.

---

## 05 Scope: what this project is and is not

### 5.1 What this project is

- A reinforcement learning research project using dengue intervention sequencing in the Philippines as the application domain.
- A CS thesis producing **two contributions**: an open, RL-ready dengue POMDP environment, and a characterization of when latent-state inference beats observation memory under state-correlated corruption.
- A proof-of-concept decision-support tool — not autonomous. The health officer stays in the loop.
- Grounded in real Philippine context, parameterized from published Philippine epidemiological literature.

### 5.2 What this project is not

- Not an autonomous public health decision system.
- Not a dengue forecasting tool.
- Not a finished product ready for DOH deployment (prospective validation, live PIDSR pipeline, per-municipality recalibration, human-factors research, certification, and a regulatory pathway are all out of scope).
- Not a proof of real-world efficacy. Validation is against historical data and simulator robustness, not deployment.
- Not a pure theory paper.

### 5.3 Design decisions — resolved

| Question | Decision | Rationale / pointer |
|---|---|---|
| Geographic scope | Single municipality — Dasmariñas, Cavite | No spatial/multi-agent complexity. Most tractable PoC. (#02) |
| Timestep | 1 week | Matches DOH reporting cadence. (#03) |
| Season length | 26 weeks | Shorter horizon = more tractable credit assignment. (#04) |
| Action space | Discrete | Trains reliably with PPO. (#05) |
| Interventions | Fogging + larviciding + LSR | Clearest SEIR-SEI mappings. (#06) |
| Observability v1 | Full (baseline only) | Need a clean full-obs baseline before measuring partial-obs cost. (#08) |
| Stochasticity | Deterministic train, stochastic eval | Fast reproducible training; robustness tested separately. (#09) |
| Budget | Single scalar | Line-item is a v2 upgrade. (#07) |
| **PoC vs thesis** | **Two regimes: PoC frozen watered-down; thesis re-hardens the full problem** | **Build first, then prove. (#11, #14)** |
| **Contribution axis** | **Partial observability (claimed); others present-not-claimed** | **Richest, product-safe, sharp novel property. (#16)** |
| **Contribution structure** | **Two contributions: open environment + inference-vs-memory characterization** | **De-risk each other. (#16)** |
| **Partial-obs method** | **Held open; world model = leading candidate, not commitment** | **Problem-first; supersedes the earlier world-model commitment. (#15)** |
| **Methods framing** | **Narrow: "observation-state coupling as the governing knob"** | **Novelty audit passed; matched-magnitude design mandated. (#17)** |

*Full rationale for every row is in `DECISIONS.md`. The research-framing decisions (#14–#17) are elaborated in `THESIS.md`.*

### 5.4 Design decisions — still open

| Question | Status |
|---|---|
| Final research-question wording | ⚠ Target drafted (`THESIS.md` §05); locked only after the existence experiment |
| Synthetic-POMDP generality commitment | ⚠ Deferred until the effect is confirmed real (`THESIS.md` §11) |
| Inference method (world model vs simpler belief method) | ⚠ Open — candidate is the world model (#15) |
| Credit assignment method | ⚠ Open — deferred axis |
| Constraint formulation (Lagrangian vs CPO) | ⚠ Start Lagrangian, compare CPO |
| Reward shaping | ⚠ Observed after baseline training |
| Explanation method | ⚠ v2, after agent works |

---

## 06 The software product

The product is **Aegis** — a web tool for a barangay/municipal health officer with no data-science background, designed so an interaction takes under five minutes and yields an actionable result. Workflow: select municipality → enter week and budget → enter case data (rainfall auto-fetched) → generate plan → receive an intervention calendar, choropleth map, projected cases averted, and a plain-language rationale.

The product is built **agent-agnostic** behind a `/recommend` contract, so a hardened observability agent later is a drop-in — no redesign. Full product spec, screens, architecture, and data model live in `PRODUCT.md`. Explainability remains a trust prerequisite (v2).

---

## 07 The RL engine

### 7.1 System overview

| Phase | What happens |
|---|---|
| **Training** | Agent interacts with the SEIR-SEI ODE simulator, learning a policy that maximizes cases averted subject to the budget constraint. Offline. |
| **Evaluation** | Trained policy tested on held-out historical seasons and perturbed simulator scenarios. |
| **Deployment** | Policy frozen, served via the web app. No continued learning from deployment (offline RL). |

### 7.2 The simulator (environment) — *a claimed contribution*

A SEIR-SEI ODE simulator as an OpenAI Gymnasium-compatible environment (`reset()`, `step()`, `observation_space`, `action_space`). This is **contribution #1** — an open, RL-ready dengue POMDP, the instrument that does not yet exist for dengue the way OpenMalaria did for malaria.

- **ODE solver:** `scipy.integrate.solve_ivp`. One `step()` = one week.
- **Parameter calibration:** Miksch et al. (2016/2019) Cebu City parameters, with documented uncertainty ranges. *Honest framing:* calibration to Dasmariñas is ill-posed (underreporting makes parameters unidentifiable), so the simulator is "literature-grounded with documented uncertainty," not "calibrated."
- **Seasonal forcing:** mosquito birth rate as a function of weekly rainfall.
- **Intervention effects:** time-varying ODE parameter perturbations (fogging drops Im briefly; larviciding cuts recruitment ~2 weeks; LSR lowers carrying capacity persistently).
- **Observation model (the contribution's experimental variable):** configurable corruption with a **state-correlation knob** — from state-independent (homoscedastic, fixed reporting fraction) to state-correlated (reporting that worsens with the true state), at controllable magnitude. v1 baseline uses full observability; the research phase sweeps this knob (see `THESIS.md` §08).

> **⚠ OPEN — Stochasticity:** deterministic for training; stochastic variants for robustness evaluation.

### 7.3 The policy network

- **Architecture v1:** MLP, 2–3 hidden layers of 256 units (full-obs baseline).
- **Memory baseline:** recurrent policy (LSTM/GRU), tuned to the Ni et al. (2022) standard.
- **Inference variant (candidate):** policy operates on an inferred latent state. The world model is the leading candidate (§07b) — *not committed*.

### 7.4 The training algorithm

**Primary:** PPO via Stable-Baselines3 — stable, handles discrete actions, directly comparable to Mai et al. (IJCAI 2023). **For the CMDP:** Lagrangian PPO. **Memory baseline:** RecurrentPPO (sb3-contrib).

> **⚠ OPEN — Credit assignment method:** deferred axis; decided only if the research phase reaches it.

### 7.5 The reward function

```
R(t) = [cases_no_intervention(t) − cases_with_intervention(t)] − cost_weight × intervention_cost(t)
```

The counterfactual `cases_no_intervention(t)` is the fixed no-intervention trajectory from season start (specified to avoid an ambiguous per-step fork). Delayed: `cases_with_intervention(t)` reflects actions 2–4 weeks prior.

### 7.6 The explanation module

Post-hoc layer applied after the policy produces its recommendation. ⚠ OPEN method (SHAP / attention / counterfactual / rule extraction); v2, after the agent works.

---

## 07b The world model component — *a candidate method, not the contribution*

*This section was originally written as the thesis's primary contribution. Following Decision #15 it is reframed: the world model is the **leading candidate method** for the inference arm of the contribution axis (§3.4), to be evaluated against a strong recurrent baseline. The **contribution is the characterization** (`THESIS.md`), not the choice of world model.*

### Why a world model is the leading candidate (memory vs inference)

A recurrent (memory) policy handles partial observability *reactively*: it remembers past noisy observations and reacts. It stays in "shadow space."

A world model handles it *actively*: it learns the dynamics — including how the true state maps to what surveillance reports — and asks "what true state is most consistent with this observation?" It infers the hidden state rather than just integrating the noisy signal. **Memory vs inference** is the central contest of the thesis.

The hypothesis (see `THESIS.md` §06): inference's advantage is largest and most reliable when the corruption is **state-correlated**, because that is where memory must learn the hardest mapping from the sparsest high-stakes data. Whether explicit inference still beats a strong recurrent baseline there is a genuinely open empirical question — which is why the method is held open, not assumed.

### Why not a full POMDP belief filter

Explicit belief-state tracking over a continuous SEIR-SEI state is analytically intractable (particle filters / variational inference are separate research areas). A learned latent state is a tractable approximation. Sufficient; not maximally precise.

### The world-model taxonomy (two distinct senses)

- **Type 1 — RL sense (Aegis uses this):** a learned network mimicking environment dynamics for inference and efficiency. Example: DreamerV3.
- **Type 2 — cognitive-science sense (adviser's research):** whether a large pretrained model has emergent physical understanding of reality. Distinct from Type 1.

### Reference

DreamerV3 (Hafner et al., 2023). More complex than SB3 PPO; requires the deep-learning/transformer foundations being built in parallel. Implementation is a research-phase task and is **contingent on the existence experiment** showing the heavier machinery is warranted. The experimental design (three agents, matched-magnitude knob) lives in `THESIS.md` §08.

---

## 08 What the literature has done

### 8.1 Tier 1 — RL for epidemic control (COVID / NPI focused)

| Paper | What they did | What they left open |
|---|---|---|
| **Mai et al., IJCAI 2023** (arXiv:2301.12802) | Multi-intervention MDP over EpiPolicy. PPO beats SAC. Dense reward, full obs, cost-in-reward. **Closest prior work; baseline to replicate.** | No vector-borne disease. No SEIR-SEI. No hard budget. No partial obs. Dense rewards. |
| **Libin et al., ECML-PKDD 2020** | PPO over influenza meta-population; school closures. | Single intervention. No budget. |
| **Colas et al., 2020 (EpidemiOptim)** | DQN / NSGA-II on SEIR COVID; on/off lockdown. | Single binary action. Full obs. |
| **Bastani et al., Nature 2021 (Eva)** | Deployed RL for COVID border testing; bandit. Real-world validated. | Bandit, not sequential MDP. |
| **Peng & Perrault, arXiv 2025** | Constrained RMAB for multi-cluster outbreak control; Lagrangian RL. | COVID only. No delayed-reward treatment. |

### 8.2 Tier 2 — RL for vector-borne diseases

| Paper | What they did | What they left open |
|---|---|---|
| **Bent et al., AAAI 2018; Makondo et al., 2021** | Malaria intervention selection as bandit/MDP; simple UCB often beats deep RL. | Malaria. Simplified sim. **Used true ODE state as reward, bypassing real delay.** |
| **KDD Cup 2019 (IBM Research-Africa)** | Malaria policy learning, 5-year horizon. **State transitions not action-dependent — weakens MDP validity.** | Malaria. Not a true sequential problem. |
| **SIT control (arXiv:2310.13072)** | RL for sterile insect technique ODE; no disease transmission. | Control-theory demo only. |

### 8.3 Tier 3 — Dengue and RL

**Finding: zero published papers combining dengue with RL for intervention sequencing** (systematic search, June 2026 — see `GAP.md`).

| Paper type | Representative works | Why it is not RL |
|---|---|---|
| Classical optimal control | Rodrigues et al. 2013; Saha & Samanta 2023 | Pontryagin; single trajectory, not policy learning. |
| Budget-constrained optimization | BMC Public Health 2021 (Thailand) | LP over dengue ODE; not RL. |
| Agent-based simulation | Miksch et al. 2016/2019 (Cebu City) | No learning agent. **Primary parameter source for Aegis.** |
| Wolbachia optimization (Philippines) | Corum et al. arXiv:2601.10967 (UP Diliman) | Multi-objective optimization, **not RL**. Closest Philippine prior art. |
| Dengue forecasting (Philippines) | medRxiv 2020; PMC3983113; PMC5905126 | Supervised forecasting; does not decide interventions. |

### 8.4 The two gaps

**Domain gap (`GAP.md`):** dengue ∩ RL ∩ intervention sequencing is empty. The entire applied project sits here.

**Methods gap (`GAP_METHODS.md` → `research/DSCOPDWIBMDRPOMDP.md`):** the *actual scientific claim* rests here — no deep-RL work isolates **state-correlation of the observation process** as the property governing the inference-vs-memory gap. Clean but narrow. Distinguished from classical belief-state theory (idealized), DVRL (noise *magnitude*), Malekzadeh & Plataniotis (noisy-vs-partial *task type*), and Cao et al. (distractor SNR). The strong recurrent baseline (Ni et al., ICML 2022) is the obstacle to beat; MNAR/informative-missingness statistics provide theoretical scaffolding.

> ⚠ The methods audit was produced by a single agent reading mostly abstracts. Verify DVRL, Ni et al., and Malekzadeh & Plataniotis before relying on the positioning.

---

## 09 Open research questions

*This section summarizes; `THESIS.md` owns the full framing, the experimental design, and the significance plan.*

### Q1 — Can a constrained RL agent outperform hand-crafted baselines on a dengue SEIR-SEI simulator? *(Required — July PoC)*

The PoC baseline question. If Lagrangian PPO with a hard budget does not beat a threshold-rule policy (the status quo), the premise has a problem. *Status: target for July, engineering phase.*

### Q2 (HEADLINE THESIS QUESTION) — Does the *structure* of observation corruption govern when inference beats memory? *(Claimed contribution)*

Does an agent that **infers** the latent state outperform one that **remembers** observations — and is the advantage governed by whether the corruption is **state-correlated**? Three-agent comparison (full-obs ceiling / strong recurrent memory / inference), with noise *magnitude held constant* while *structure* is toggled. The gate is the existence experiment (`THESIS.md` §08).

*Status: target wording drafted, **not locked** until the existence experiment confirms the phenomenon. Method held open; world model leading candidate.*

### Q3 — How robust is the trained policy to ODE parameter misspecification?

Train on the literature-grounded ODE; evaluate on perturbed variants. A **supporting** contribution that converts the ill-posed-calibration liability into a robustness result. *Status: research phase; pairs with Q2.*

### Q4 — Credit assignment under delayed reward *(deferred axis)*

Compare discounting vs RUDDER vs HER vs world-model imagination. *Status: deferred; the short horizon may make it mild.*

### Q5 — Can recommendations be explained in terms a health officer finds actionable? *(v2)*

Qualitative (domain experts) and quantitative (feature attribution). *Status: v2, after the agent works.*

> **Spatial ("where")** is framed in `THESIS.md` as the second-richest axis but is **deferred** (hierarchical/multi-agent RL; would change the product). Present in the environment; not claimed.

---

## 10 Project timeline

### Phase 1 — Engineering PoC (Now to July)

**Goal:** a working proof of concept in the watered-down regime. Not optimal, not rigorously validated — real enough to show what Aegis is becoming. **Deliverables:** SEIR-SEI Gym environment (full-obs baseline), basic PPO agent, the surface web tool, an honest demo. **Not required by July:** optimal agent, rigorous constrained RL, world model, multi-seed validation. *This is an engineering deadline, not a research deadline.* Build plan: `CURRICULUM.md`.

### Phase 2 — Research (July to Submission)

**Goal:** the two contributions, made rigorous.

1. **The open environment** — harden the PoC simulator into a documented, RL-ready POMDP with the state-correlation observation knob (contribution #1; Datasets & Benchmarks-shaped).
2. **The characterization** — the inference-vs-memory study (Q2), gated by the existence experiment first, then the matched-magnitude sweep on the synthetic POMDP *and* the dengue environment (generality).

**Sequencing discipline:** the cheap existence experiment runs *before* committing to the world model or the synthetic-environment build. Then Q3 (robustness) as support. The PoC is the laboratory; the thesis is the analysis of what re-hardening the observability axis reveals.

### Parallel study track

- Karpathy Zero to Hero / nanoGPT — deep-learning and transformer foundations (prerequisite *if* the world model is chosen).
- DreamerV3 paper and codebase — world-model preparation, as a **candidate**.
- Sutton & Barto Ch. 1–6 — RL foundations.
- POMDP / belief-state literature and the methods-novelty anchors (§11.3) — for the contribution framing.

---

## 11 Reading list

### 11.1 Foundations

| Resource | Why |
|---|---|
| Sutton & Barto, Ch. 1–3 | MDP, Bellman, value function. `incompleteideas.net/book/the-book.html` |
| OpenAI Spinning Up, Parts 1–2 | Policy gradients and PPO. `spinningup.openai.com` |
| Gymnasium docs | Standard RL environment interface. `gymnasium.farama.org` |
| Keeling & Rohani, Ch. 1–3 | Epidemiology incl. vector-borne host-vector models. |

### 11.2 Prior work (domain)

| Paper | Why |
|---|---|
| Mai et al., IJCAI 2023 (arXiv:2301.12802) | Backbone to extend; PPO over EpiPolicy. |
| Miksch et al. 2016/2019 — Cebu City | Primary parameter source. |
| Corum et al., arXiv:2601.10967 — UP Diliman | Closest Philippine dengue-intervention prior art. |
| Peng & Perrault, arXiv:2603.19397 | Best CMDP public-health RL; Lagrangian reference. |
| Altman, *Constrained MDPs*, 1999 | CMDP foundation (Ch. 1–3). |

### 11.3 The contribution: partial observability, inference vs memory *(verify before relying — §08.4)*

| Paper | Why |
|---|---|
| Ni, Eysenbach & Salakhutdinov, ICML 2022 (arXiv:2110.05038) | The strong recurrent baseline — the bar to beat. |
| Igl et al. (DVRL), ICML 2018 (arXiv:1806.02426) | Inference helps with noise *magnitude* — distance the claim from this. |
| Malekzadeh & Plataniotis (arXiv:2212.07946) | Inference vs recurrent; *task type*, not structure — distance from this. |
| Kaelbling, Littman & Cassandra, *Artif. Intell.* 1998 | Belief-state sufficiency (idealized theory) — distance from this. |
| MNAR / informative-missingness statistics | Theoretical scaffolding: "inference helps iff missingness is informative." |
| Hafner et al. 2023 — DreamerV3 | Leading candidate inference method (not committed). |

### 11.4 Data sources

DOH epi bulletins (weekly cases); PAGASA (rainfall/temperature); PSA 2020 Census (denominators); HDX (`data.humdata.org`, "Philippines dengue"); PhilGIS / NAMRIA (boundaries). *PIDSR granular data requires institutional access — future work.*

---

## 12 Glossary

| Term | Plain language definition |
|---|---|
| **Action space** | The set of actions available each timestep. Discrete in Aegis v1. |
| **Agent** | The RL learner. Observes, acts, updates policy from reward. |
| **Belief state** | The agent's distribution over possible true states given observations. Inferred (approximately) by the inference policy. |
| **Bellman equation** | Value of state = immediate reward + discounted future value. |
| **CMDP** | Constrained MDP: an MDP with hard constraints on cumulative cost. |
| **Credit assignment** | Determining which past action deserves credit for a delayed reward. |
| **DreamerV3** | World-model RL algorithm (Hafner et al. 2023). Leading *candidate* inference method for Aegis. |
| **Inference (vs memory)** | Reconstructing the hidden true state from observations using learned dynamics, rather than just remembering past observations. The thesis's central contest. |
| **Memory (vs inference)** | Conditioning the policy on a history of observations (e.g. a recurrent/LSTM policy). |
| **MDP / POMDP** | (Partially Observable) Markov Decision Process — the formal frameworks; POMDP = agent sees noisy/incomplete observations. |
| **PPO** | Proximal Policy Optimization. Primary algorithm; stable, discrete-action, comparable to Mai et al. |
| **Recurrent baseline** | A memory-based policy (RecurrentPPO) — the bar the inference method must beat (Ni et al. 2022). |
| **SEIR-SEI** | Dengue host-vector model: SEIR humans + SEI mosquitoes, coupled through biting rate. |
| **State-correlated (vs state-independent) corruption** | Observation distortion whose *structure* depends on the hidden state (e.g. reporting worsens during surges) vs. distortion independent of it. The property at the heart of the thesis claim. |
| **World model (Type 1)** | A learned network of environment dynamics enabling hidden-state inference and imagination. The candidate inference method. |

---

*End of AEGIS.md · The umbrella document. Research-challenge detail lives in `THESIS.md`; methods novelty in `GAP_METHODS.md`; the decision log in `DECISIONS.md`. When something is decided, update it and mark it ✓ DECIDED with rationale; when held open, mark it ⚠ OPEN. The document is more useful with honest uncertainty than with premature answers.*