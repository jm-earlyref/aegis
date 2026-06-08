# AEGIS
### Reinforcement Learning for Vector-Borne Disease Intervention Sequencing

*A shield. A vector. A learning agent.*

---

> *Aegis (n.) — a shield or protection. From the Greek aigis, the protective armor of the gods.*
> *Aedes aegypti — the mosquito that carries dengue.*
> *The name is not a coincidence.*

---

*Primary research reference — living document, not a thesis draft.*
*Project by a 3rd year CS undergrad, Dasmariñas, Calabarzon, Philippines.*
*Last updated: June 2026*

---

## 00 How to use this document

This is `AEGIS.md` — the primary source of truth for the Aegis project. It is not a thesis draft, a proposal, or a literature review. Its job is to keep your thinking organized as the project evolves — so you can return to it after a week away from the work and immediately remember what you are building, why, and where you are.

Every section has a distinct purpose. Read **section 01** when you need to re-ground yourself emotionally. Read **sections 02–04** when you need to explain or remember the technical structure. Read **section 05** when preparing related work. Read **section 06** when designing experiments. Read **section 07** when choosing what to study next. Read **section 08** when you forget a term mid-read.

**What is marked ⚠ OPEN** is an explicit design decision that has not yet been made. Do not treat these as gaps to fill in immediately — they are honest placeholders. The document is more useful with honest uncertainty than with premature answers.

---

## 01 The north star

Almost every Filipino family has a dengue story. In 2024, the Philippines reported 413,960 dengue cases — the highest in ASEAN — with Calabarzon, your home region, among the most affected. This is not an abstract public health statistic. It is the reason this project exists.

The image that grounds this work is concrete: a barangay health officer, on a Monday morning, deciding where to deploy fogging, larviciding, or community source reduction. They have incomplete case reports from last week, a rough rainfall forecast, and ₱50,000 left for the month. Whatever they decide, they will not see the result for two to four weeks. Right now, they make that decision using intuition, local knowledge, and fixed schedules. There is no tool that tells them where to act and when — and actually works.

That is the problem. Not dengue forecasting. Not epidemiological modeling. Not academic benchmarking. The problem is: can we build something that earns that health officer's trust?

> *Can we build an RL agent that learns intervention policies good enough to be trusted by a real health officer, trained on the kind of incomplete, noisy, resource-constrained reality they actually live in?*

The research novelty is not the domain. It is what you have to invent to make the agent work under the three simultaneous constraints that define the health officer's reality: delayed feedback, incomplete information, and a hard budget limit. Those three things together, in a vector-borne disease setting, have never been tackled in the literature. The dengue domain is the vehicle. The method that emerges from solving it is the contribution.

This project is inspired by the AlphaFold model of research: pick a real problem that matters, understand it deeply enough to see what current tools cannot do, and build whatever is necessary. The theory follows the problem, not the other way around.

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

**SEIR-SEI** (pronounced as letters: S-E-I-R, S-E-I) is the standard compartmental model for dengue. It tracks two interacting populations — humans and mosquitoes — each partitioned into disease stages. "Compartmental" means every individual is assumed to be in exactly one stage at a time, and the model tracks how many individuals move between stages over time.

#### Human compartments (SEIR)

| Compartment | Meaning |
|---|---|
| **S — Susceptible** | Healthy humans with no immunity. Can be infected by a bite from an infectious mosquito. |
| **E — Exposed** | Infected but not yet infectious. The virus is replicating. Duration: approximately 5–7 days. Also called the intrinsic incubation period. |
| **I — Infectious** | Sick and capable of infecting a mosquito that bites them. Duration: approximately 5–7 days. This is when symptoms appear. |
| **R — Recovered** | Immune to the specific dengue serotype that infected them. There are four serotypes (DENV-1 to DENV-4); recovery from one does not guarantee immunity to others. |

#### Mosquito compartments (SEI)

Mosquitoes use a simplified three-compartment model because they do not recover — once infectious, an *Aedes* mosquito remains infectious until it dies.

| Compartment | Meaning |
|---|---|
| **Sm — Susceptible mosquitoes** | Healthy adult mosquitoes capable of being infected. Population size is driven by rainfall and temperature: more rain → more standing water → more breeding sites → more mosquitoes. |
| **Em — Exposed mosquitoes** | Mosquitoes that have bitten an infectious human and are now carrying the virus but cannot yet transmit it. Duration: approximately 8–12 days. Also called the extrinsic incubation period. |
| **Im — Infectious mosquitoes** | Mosquitoes capable of infecting humans. They remain in this state until they die (adult mosquito lifespan: approximately 2–4 weeks). |

#### How the two populations interact

The coupling between the human and mosquito models is through the biting rate. The rate at which susceptible humans become exposed (S → E) depends on how many infectious mosquitoes there are. The rate at which susceptible mosquitoes become exposed (Sm → Em) depends on how many infectious humans there are. This bidirectional coupling is what makes dengue hard to control — you have to manage both populations simultaneously.

### 2.3 How interventions map to the model

Each intervention type targets a different compartment or parameter. Understanding this mapping is essential for designing the action space of the RL agent.

| Intervention | What it targets | Effect in the model |
|---|---|---|
| **Fogging (space spraying)** | Im — adult mosquitoes | Reduces Im directly. Fast-acting but short-lived (days). Does not affect larvae or eggs. |
| **Larviciding** | Pre-Sm — larvae | Reduces the recruitment rate into Sm. Effect appears 1–2 weeks later as larvae die before reaching adulthood. |
| **Larval source reduction (LSR)** | Breeding sites | Reduces the carrying capacity of the mosquito population. Removes standing water. Effect accumulates over weeks. |
| **Wolbachia release** | Sm and Im | Releases Wolbachia-infected mosquitoes that suppress dengue transmission. Long-horizon, irreversible. Still experimental at scale in the Philippines. |
| **Vaccination (TAK-003, Dengvaxia)** | S humans | Moves susceptible humans to a protected state. Serotype-specific coverage. Requires prior serosurvey. |
| **Community campaigns** | Breeding behavior | Reduces household-level breeding sites through behavior change. Hard to quantify. Effect is diffuse and delayed. |

The key observation: no single intervention is sufficient. Fogging reduces adult mosquitoes fast but the population rebounds within days if breeding sites are untouched. Larviciding and LSR take weeks to show effect. A rational strategy sequences them — but the right sequence depends on the current state of both the human and mosquito populations, the time in the season, and the remaining budget. That sequencing problem is precisely what the RL agent must learn.

---

## 03 The formal problem structure

### 3.1 What is a Markov Decision Process (MDP)?

**Markov Decision Process (MDP)** — the mathematical framework for any problem where an agent makes sequential decisions, each affecting the future, with the goal of maximizing cumulative reward over time.

An MDP has five components. Every RL problem is formally an MDP. The dengue problem maps cleanly onto all five.

| MDP Component | Plain meaning |
|---|---|
| **State (S)** | Everything the agent can observe about the world at a given moment. |
| **Action (A)** | The set of things the agent can do at each step. |
| **Reward (R)** | A scalar signal the agent receives after taking an action. The agent's goal is to maximize the sum of rewards over time. |
| **Transition function (T)** | The rules that govern how the world changes in response to an action. In our case, this is the SEIR-SEI ODE simulator. |
| **Discount factor (γ)** | A number between 0 and 1 that weights future rewards less than immediate ones. γ = 0.99 means a reward 100 steps away is worth 0.99^100 ≈ 0.37 of a reward now. |

### 3.2 The dengue problem as an MDP

#### State

The state is everything the agent observes at the start of each decision timestep. In the dengue setting this includes:

- Reported case counts by area for the past N weeks *(note: observed, not true — see section 3.4 on partial observability).*
- Current week of the year (proxy for season and expected rainfall-driven mosquito dynamics).
- Rainfall index for the current and forecast week.
- Remaining budget for the season.
- What interventions were deployed in recent weeks and where (the agent needs to know, for example, that it fogged Barangay A last week so it can account for the rebound effect).

> **⚠ OPEN — State representation:** Should the state include raw ODE compartment values (S, E, I, R counts) or only observable proxies (reported cases, rainfall)? The former is full observability and is unrealistic. The latter is partial observability and is the real problem. See section 3.4.

#### Action

The action is the agent's deployment decision for the current week. The action space needs to encode: which intervention type, which geographic area(s), and at what intensity or budget allocation.

> **⚠ OPEN — Action space structure:** There are two main choices: (1) **Discrete** — the agent selects from a fixed menu of intervention-area combinations each week. Simpler to train, less expressive. (2) **Continuous** — the agent outputs a budget allocation vector across intervention types and areas. More realistic, harder to train (requires SAC or continuous PPO). This choice affects the algorithm selection significantly.

#### Reward

**Reward** — the signal the agent receives after taking an action that tells it how well it did.

In this problem the natural reward is cases and deaths averted relative to a no-intervention or fixed-schedule baseline. The critical challenge: this reward is delayed. An intervention deployed in week 3 may not produce a measurable case reduction until week 6 or 7. The agent acts now and receives feedback weeks later.

> **⚠ OPEN — Reward shaping:** Should intermediate rewards be added (e.g., reward for reducing larval index counts, which are observable faster than case reduction) to help the agent learn? This is called reward shaping. The risk is introducing bias — the agent may optimize the proxy instead of the true goal. This is a known problem in RL called **reward hacking**.

#### Transition function

The transition function is the SEIR-SEI ODE simulator. The agent takes an action, the simulator steps forward by one timestep, and the new state is returned. The simulator is the environment.

> **⚠ OPEN — Timestep:** What is one decision step? One week is the natural choice (aligned with how health officers plan and with the typical DOH reporting cadence). One day is more granular but may be too fine for meaningful intervention decisions. Two weeks is coarser but may match some field realities better. This choice affects the total horizon length and the severity of the delayed reward problem.

> **⚠ OPEN — Simulator fidelity:** ODE (fast, analytically tractable, wrong in detail) vs agent-based model or ABM (realistic, expensive, hard to train on). Strong prior recommendation from the literature: train the RL agent on an ODE simulator and validate the trained policy on an ABM or real data. Do not train on an ABM — it is too slow.

### 3.3 Constrained MDP (CMDP)

**Constrained MDP (CMDP)** — an MDP with one or more hard constraints that the agent must satisfy in addition to maximizing reward.

Standard RL folds cost into the reward as a penalty — the agent can overspend if it calculates that the reward justifies it. A real health officer cannot do this. The budget is a hard limit, not a preference. This transforms the problem from an MDP into a CMDP.

The formal CMDP adds a cost function C(s, a) and a budget threshold B. The agent must maximize expected cumulative reward subject to the constraint that expected cumulative cost does not exceed B.

> **⚠ OPEN — Constraint formulation:** Three main approaches: (1) **Lagrangian relaxation** — add a penalty term λ·C to the reward and tune λ. Simple to implement, unstable to train. (2) **Constrained Policy Optimization (CPO)** — guarantees constraint satisfaction during training but is complex to implement. (3) **Primal-dual methods** — a middle ground. For a thesis, Lagrangian PPO is the most tractable starting point, with CPO as a comparison.

### 3.4 Partial observability (POMDP)

**Partially Observable MDP (POMDP)** — an MDP where the agent cannot directly observe the true state of the world — only a noisy, incomplete observation derived from the true state.

Standard epidemic RL assumes the agent can see the true compartment values. This is unrealistic. In the Philippines, approximately 1 in 4 dengue cases is reported to surveillance. The agent never sees the true I count — it sees a fraction of it, with a delay of 1–2 weeks.

This partial observability has practical consequences: the agent must act on a signal that underestimates the true severity of the epidemic. A policy trained on full observability will perform worse when deployed on real data than its simulator metrics suggest.

> **⚠ OPEN — How to handle partial observability:** Three options: (1) Ignore it in the first version — train with full observability as a baseline, note the gap as a limitation. (2) Add observation noise to the simulator to approximate underreporting — train with a recurrent policy (LSTM or GRU) that integrates observations over time. (3) Full POMDP formulation with explicit belief-state tracking. Option 1 is the fastest path; option 2 is the most publishable for a thesis; option 3 is the most technically ambitious.

---

## 04 The three hard things

This section explains why the dengue intervention RL problem is not just "RL applied to a new domain." Each difficulty below is individually studied in the RL literature. The combination of all three, in a vector-borne disease setting, has never been tackled. That combination is the research contribution.

### 4.1 Delayed reward and credit assignment

**Delayed reward** — a setting where the reward signal arrives many timesteps after the action that caused it.

**Credit assignment problem** — the challenge of determining which past action in a trajectory deserves credit (or blame) for a reward received later.

In the dengue setting, the agent fogs Barangay A in week 3. Case counts begin dropping in week 6. Three weeks of other decisions sit between the action and the outcome. Standard RL methods (PPO, SAC) backpropagate credit using Bellman equations, which work well when reward is dense but struggle when reward is sparse and delayed.

The credit assignment problem is compounded by the fact that multiple interventions are deployed simultaneously across multiple areas. Even if the agent can detect that cases dropped in week 6, it cannot easily attribute that drop to the fogging in week 3, the larviciding in week 2, or the community campaign in week 1.

**Relevant methods:** RUDDER (return decomposition), Hindsight Experience Replay (HER), reward redistribution, and learned world models (DreamerV3). None of these have been applied to epidemic control. This is an open experimental question for the thesis.

### 4.2 Partial observability

Addressed formally in section 3.4. The practical consequence: the agent's state is always an underestimate of the true epidemic state. Roughly 75% of dengue cases are never reported. Reporting also has a lag of 1–2 weeks. The agent is always flying partially blind.

This matters beyond just performance degradation. It affects trust: a health officer who knows the data is incomplete will be more skeptical of recommendations that appear to rely on data they know is wrong. The PoC must communicate uncertainty explicitly.

### 4.3 Hard budget constraints

Addressed formally in section 3.3. The key distinction to hold: in most RL, cost is a soft penalty folded into the reward function. The agent trades off between reward and cost dynamically. In the health officer's world, the budget is a hard constraint — there is no amount of cases-averted reward that justifies overspending the monthly allocation. The agent must learn to operate within the constraint at all times, not just on average.

Constrained RL is a well-developed subfield (CMDP theory dates to Altman 1999), but it has rarely been applied with the combination of long-horizon planning and delayed reward that characterizes the dengue problem.

### 4.4 Why the combination is the gap

| Problem | Studied in isolation? | Studied in epidemic RL? |
|---|---|---|
| Delayed reward | Yes — extensively | No — all epidemic RL uses dense rewards |
| Partial observability | Yes — POMDP literature | No — all epidemic RL assumes full observability |
| Hard budget constraints | Yes — CMDP literature | Partially — only for COVID resource allocation |
| Vector-borne SEIR-SEI dynamics | Yes — epidemiology | No — all epidemic RL models respiratory disease |
| All four together | No | No |

The thesis does not need to solve all four simultaneously in full generality. A clean empirical study of how they interact — which combinations degrade performance most, which methods address which problems — is itself a publishable contribution that generalizes beyond dengue.

---

## 05 Scope: what this project is and is not

### 5.1 What this project is

- A reinforcement learning research project using dengue fever intervention sequencing in the Philippines as the primary application domain.
- A CS thesis producing both a methodological contribution (how to train RL agents under delayed reward, partial observability, and hard budget constraints) and an applied demonstration (a working tool for health officers).
- A proof-of-concept decision-support tool — not an autonomous system. The health officer remains in the loop. The agent recommends; the officer decides.
- A project grounded in real Philippine context: DOH/PIDSR surveillance data, Calabarzon epidemiological parameters, and the actual resource constraints of LGU-level public health.
- A stepping stone toward the AlphaFold model of applied RL research: problem-first, method-as-necessary, contribution-as-discovery.

### 5.2 What this project is not

- Not an autonomous public health decision system. No recommendations are deployed without human review.
- Not a dengue forecasting tool. Forecasting predicts risk. This project decides what to do about risk.
- Not a national-scale simulation. The PoC targets a manageable geographic scope (see open design questions below).
- Not a proof of real-world efficacy. The thesis validates against historical data and simulator robustness tests — not a randomized controlled trial.
- Not a pure theory paper. The theoretical contribution is grounded in and demonstrated through the dengue domain.
- Not a finished product ready for DOH deployment. It is a research prototype with an interface demonstrating what a production tool could look like.

### 5.3 Open design questions on scope

> **⚠ OPEN — Geographic scope:** What is the unit of geographic resolution? Options: (a) Single municipality or city (e.g., Dasmariñas, Cavite) — simplest, most tractable, best for a thesis PoC. (b) Province-level (e.g., all of Cavite) — requires spatial structure in the MDP, possibly multi-agent formulation. (c) Region-level (Calabarzon) — significantly increases complexity, may require hierarchical RL. Recommendation: start with a single municipality and validate thoroughly before expanding.

> **⚠ OPEN — Timestep resolution:** One week is the natural default. Daily is more granular but increases the horizon length and worsens the delayed reward problem. Bi-weekly may better match field reporting. This should be decided in consultation with what the Philippine DOH/PIDSR data actually provides.

> **⚠ OPEN — Season length:** How many timesteps is one "season"? Dengue in the Philippines peaks June–October (rainy season). A 52-week annual horizon captures full seasonality but is a long RL horizon. A 26-week season (April–September) may be more tractable. This is a training and curriculum design question as much as an epidemiological one.

> **⚠ OPEN — Number of interventions in the action space:** Starting with 2–3 intervention types (fogging, larviciding, LSR) is more tractable than including vaccines and Wolbachia release immediately. The latter two have different time horizons, data requirements, and policy implications. A clean thesis could focus on vector-control interventions only and leave biological control as future work.

> **⚠ OPEN — Budget granularity:** Is the budget a single scalar (total pesos per month) or a multi-category constraint (personnel budget, supplies budget, logistics budget separately)? The scalar version is more tractable and defensible as an initial simplification, but a real health officer manages line-item budgets.

---

## 06 The software product

### 6.1 What the health officer interacts with

The product is **Aegis** — a web application (browser-based, no installation required, shareable via URL). It is designed for a barangay or municipal health officer with no data science background. The interface must be simple enough that the interaction takes under five minutes and produces a result the officer can act on immediately.

**The health officer's workflow:**

1. Open the web app on any device (laptop, tablet, mobile).
2. Select their region or municipality from a dropdown (pre-loaded with Philippine administrative boundaries).
3. Input the current week of the year and available budget for the season.
4. The system pulls the latest available case data for their area (from DOH/PIDSR or manually entered).
5. Click "Generate plan."
6. Receive a recommended intervention calendar: which intervention to deploy, in which barangay, in which week, for the next N weeks.
7. See the projected cases averted relative to a fixed-schedule baseline, with a confidence range.
8. For each recommendation, see a plain-language explanation of why — which factor drove the recommendation (e.g., "High rainfall forecast for weeks 4–6 suggests elevated larval population; LSR now will reduce adult mosquito emergence before peak transmission").
9. Download the plan as a printable PDF or share the URL with a supervisor.

**Explainability requirement:** The officer must be able to understand why each recommendation was made. This is not optional — it is a prerequisite for trust. The agent's output must be accompanied by a readable summary of the top contributing factors. This constrains the method: highly opaque models without an explanation layer are insufficient.

### 6.2 The data IPO model

#### Inputs

| Input | Source | Notes |
|---|---|---|
| Weekly dengue case counts by area | DOH PIDSR / HDX / manual entry | Aggregated at province or municipality level. Approximately 1 in 4 true cases reported. 1–2 week lag. |
| Rainfall data | PAGASA forecast / historical | Weekly rainfall index. Strong driver of *Aedes* breeding cycle. |
| Temperature data | PAGASA | Affects mosquito development rate and lifespan. |
| Available budget | Health officer input | Total pesos available for the season or current month. |
| Previous interventions deployed | Health officer input or system memory | What was deployed in the past N weeks and where. Needed for rebound effect modeling. |
| Administrative boundaries | PSA shapefiles / PhilGIS | Barangay or municipality polygons for spatial structure. |
| Population denominators | PSA 2020 census | Required for incidence rates (cases per 100,000). |

#### Process — the RL engine (see section 07 for full detail)

- The input data is assembled into the current state vector for the MDP.
- The trained RL policy takes the state and outputs an action (intervention allocation).
- The policy is evaluated for the next N timesteps via rollout through the SEIR-SEI simulator.
- The output is a multi-week intervention calendar with projected case trajectories.
- The explanation module reads the policy's attention or gradient signal to generate the plain-language rationale.

#### Outputs

| Output | Format | For whom |
|---|---|---|
| Recommended intervention calendar | Week-by-week table: intervention type, area, intensity | Health officer — primary output |
| Projected cases averted | Number + confidence range vs baseline | Health officer — for justification to supervisors |
| Plain-language rationale per recommendation | 1–2 sentences per week's recommendation | Health officer — for trust and transparency |
| Uncertainty flags | Highlighted weeks where confidence is low | Health officer — for caution |
| Downloadable PDF plan | Printable A4 summary | For filing, sharing, supervisor approval |
| Model assumptions summary | Footnote / expandable section | For any technically-minded reviewer |

---

## 07 The RL engine

This section describes the core technical component of the system — the trained RL agent that generates intervention recommendations. It is written as a semantic reference: what each component is, what it does, and how it connects to the others.

### 7.1 System overview

The RL engine has three phases: training, evaluation, and deployment. These are distinct and should be kept conceptually separate.

| Phase | What happens |
|---|---|
| **Training** | The agent interacts with the SEIR-SEI ODE simulator thousands of times, learning a policy that maximizes cases averted subject to the budget constraint. This happens offline, before the product is deployed. |
| **Evaluation** | The trained policy is tested on held-out historical seasons and on simulator scenarios with perturbed parameters to assess robustness. The output is a set of performance metrics. |
| **Deployment** | The trained policy is frozen and served via the web application. It takes real observed data as input and outputs recommendations. It does not continue learning from deployment interactions (offline RL). |

### 7.2 The simulator (environment)

**Environment** — in RL, the world that the agent interacts with. The environment receives actions and returns new states and rewards.

The environment is a SEIR-SEI ODE simulator implemented as an OpenAI Gymnasium-compatible environment. This means it has a standard interface (`reset()`, `step()`, `observation_space`, `action_space`) that any standard RL library (Stable-Baselines3, CleanRL) can interact with.

- **ODE solver:** The simulator steps forward by integrating the SEIR-SEI differential equations numerically (e.g., using `scipy.integrate.odeint` or a fixed Euler step). One call to `step()` advances the simulation by one timestep (tentatively one week).
- **Parameter calibration:** The ODE parameters (transmission rates, incubation periods, mosquito lifespan, seasonal forcing) are calibrated to Philippine dengue data. Starting point: Miksch et al. (2016) Cebu City ABM parameters, adjusted for Calabarzon using DOH/PIDSR case data.
- **Seasonal forcing:** The mosquito birth rate is a function of rainfall — parameterized using historical PAGASA weekly rainfall data for the target region. This drives the annual epidemic cycle.
- **Intervention effects:** Each intervention type modifies a specific ODE parameter for a specified duration and magnitude. Fogging reduces Im by a factor α for τ days. Larviciding reduces the larval recruitment rate by β for τ days. These are modeled as time-varying parameter perturbations.
- **Observation function:** In the partial observability setting, the simulator does not return true compartment values. It returns a noisy, lagged observation that simulates the DOH reporting process: true I values are multiplied by a reporting fraction r ≈ 0.25 and delayed by 1–2 weeks.

> **⚠ OPEN — Stochasticity:** Should the ODE be deterministic (same parameters → same trajectory every time) or stochastic (demographic noise, random events)? Deterministic is faster and easier to train on. Stochastic is more realistic and tests robustness. A reasonable path: train on deterministic, evaluate on stochastic.

### 7.3 The policy network

**Policy** — a function that maps states to actions (or to probability distributions over actions). The RL agent's policy is what is learned during training and what is used at deployment time.

The policy network is a neural network that takes the current state vector as input and outputs either a probability distribution over discrete actions or a mean and variance over continuous actions.

- **Architecture:** A multi-layer perceptron (MLP) with 2–3 hidden layers of 256 units each is the standard starting point for vector state spaces. If partial observability is included and the agent needs to remember past observations, a recurrent architecture (LSTM or GRU) is used instead — the hidden state acts as the agent's memory.
- **Input:** The state vector (case counts, rainfall, budget remaining, week of year, recent intervention history). Normalized to zero mean and unit variance before input.
- **Output (discrete):** A softmax distribution over all intervention-area-intensity combinations. The agent samples during training (exploration) and takes the argmax at deployment (exploitation).
- **Output (continuous):** A mean vector and diagonal covariance matrix over budget allocation fractions. Sampled from a Gaussian during training; mean used at deployment.

### 7.4 The training algorithm

**Proximal Policy Optimization (PPO)** — a policy gradient RL algorithm that updates the policy in small, stable steps by clipping the gradient update. It is the most widely used algorithm for continuous control tasks and is the recommended starting point for this project.

PPO is chosen as the primary algorithm for three reasons: it is stable to train, it works for both discrete and continuous action spaces, it has strong open-source implementations (Stable-Baselines3, CleanRL), and it is the algorithm used by Mai et al. (IJCAI 2023) — the closest prior work — making direct comparison easier.

**Soft Actor-Critic (SAC)** — an off-policy actor-critic algorithm that maximizes both reward and entropy (randomness) in the policy. More sample-efficient than PPO but more complex to tune. Used as a comparison baseline.

**For the constrained MDP:** Lagrangian PPO extends PPO by adding a Lagrange multiplier λ that penalizes budget constraint violations. The multiplier is updated each training step via gradient ascent — if constraints are violated, λ increases, making the penalty larger.

> **⚠ OPEN — Credit assignment method:** Which method (if any) is added to handle delayed reward? Options: (1) Standard PPO with reward discounting — simplest, may be sufficient if γ is tuned well. (2) Reward redistribution (RUDDER) — decomposes the return across the trajectory to redistribute credit. (3) Hindsight Experience Replay (HER) — relabels failed trajectories as successes for alternative goals. The choice here is a primary experimental variable of the thesis.

### 7.5 The reward function

The reward at each timestep is (tentatively) the number of new dengue cases prevented relative to the no-intervention counterfactual, minus an intervention cost term:

```
R(t) = [cases_no_intervention(t) − cases_with_intervention(t)] − cost_weight × intervention_cost(t)
```

The delayed nature of this reward is the central challenge: `cases_with_intervention(t)` reflects the consequence of actions taken 2–4 weeks prior, not the current action.

> **⚠ OPEN — Dense vs sparse reward:** The above reward is computed every timestep, making it technically dense. But because the reward at step t reflects actions from steps t-3 to t-1, it is effectively sparse from the credit assignment perspective. Whether to add truly intermediate rewards (e.g., a reward for reducing larval count) is an open design question with risk of reward hacking.

### 7.6 The explanation module

The explanation module is a post-hoc layer that reads the trained policy's behavior and generates the plain-language rationale shown to the health officer. This is not part of the RL training loop — it is applied after the policy has produced its recommendation.

> **⚠ OPEN — Explanation method:** Options include: (1) SHAP values applied to the policy network inputs — shows which state features most influenced the action. (2) Attention weights if a transformer-based policy is used. (3) Counterfactual explanation — "If rainfall were lower, the agent would have recommended X instead of Y." (4) Simple rule extraction — fit a decision tree to the policy's input-output pairs and display the tree's logic. Method 4 is the most interpretable to a non-technical officer; method 1 is most standard in ML explainability literature.

---

## 08 What the literature has done

### 8.1 Tier 1 — RL for epidemic control (COVID / NPI focused)

The epidemic RL literature is dominated by COVID-19 non-pharmaceutical interventions — lockdowns, school closures, vaccination timing. These papers establish the methodological baseline that this thesis extends.

| Paper | What they did | What they left open |
|---|---|---|
| **Mai et al., IJCAI 2023** (arXiv:2301.12802) | Multi-intervention MDP over EpiPolicy simulator. PPO beats SAC on 6 environments. Dense reward, full observability, cost-in-reward (no hard budget constraint). **The closest prior work.** | No vector-borne disease. No SEIR-SEI host-vector model. No hard budget constraint. No partial observability. Dense rewards throughout. |
| **Libin et al., ECML-PKDD 2020** | PPO over 379-patch influenza meta-population (Great Britain). School closures as discrete actions. Multi-region structure. | Single intervention type. Influenza, not arbovirus. No budget constraint. |
| **Colas et al., 2020 (EpidemiOptim)** | OpenAI Gym SEIR COVID model. DQN and NSGA-II. On/off lockdown trading death toll vs economic recession. | Single binary action. Dense reward. Full observability. |
| **Bastani et al., Nature 2021 (Eva)** | Deployed RL for COVID border testing in Greece. Bandit formulation. Real-world validated — identified 1.85× more cases than random surveillance. | Bandit, not sequential MDP. Testing allocation only. |
| **Peng & Perrault, arXiv 2025** | Constrained RMAB for multi-cluster outbreak control. Hierarchical Lagrangian RL. Beats baselines by 20–30%. Closest CMDP public health RL. | COVID only. Not vector-borne. No delayed reward treatment. |

### 8.2 Tier 2 — RL for vector-borne diseases

Vector-borne RL exists almost entirely for malaria, driven by IBM Research-Africa. No Aedes-borne disease (dengue, Zika, chikungunya, West Nile) has been addressed with RL.

| Paper | What they did | What they left open |
|---|---|---|
| **Bent et al., AAAI 2018; Makondo et al., arXiv 2021** | Malaria intervention selection as bandit/MDP. Interventions: ITN bed nets and IRS spraying. Key finding: simple UCB algorithms often beat deep RL on the malaria Gym environment. | Malaria, not dengue. Simplified simulator. Bandits, not deep RL. No budget constraint. No partial observability. |
| **KDD Cup 2019 (IBM Research-Africa)** | Malaria policy learning competition. 5-year horizon. Key simplification: state transitions are NOT action-dependent — weakens the MDP validity. | Malaria only. Simplified non-MDP structure. No deep RL required. |
| **SIT control (arXiv:2310.13072)** | RL applied to sterile insect technique ODE for mosquito population control. No disease transmission modeled. | No disease — only mosquito population dynamics. Control theory demonstration only. |

### 8.3 Tier 3 — Dengue and RL

**Finding: there are no published papers combining dengue with reinforcement learning for intervention sequencing.**

This is a confirmed negative result, not an assumption. The dengue control literature uses classical optimal control (Pontryagin maximum principle), agent-based simulation with hand-crafted fixed strategies, or ML forecasting. None of these involve a learning agent making sequential decisions.

| Paper type | Representative works | Why it is not RL |
|---|---|---|
| Classical optimal control | Rodrigues et al. 2013; Saha & Samanta 2023; Rawson et al. 2020 | Pontryagin maximum principle. Solves for optimal time-varying controls analytically. Single trajectory, not policy learning. |
| Budget-constrained optimization (non-RL) | BMC Public Health 2021 (Thailand) | Mathematical optimization (LP) over a dengue ODE. Closest prior art on budget constraint — but not RL. |
| Agent-based simulation | Miksch et al. 2016/2019 (Cebu City); Pascoe et al. 2024 (Dar es Salaam) | Simulates outcomes of pre-specified strategies. No learning agent. |
| Wolbachia optimization (Philippines) | Corum et al. arXiv:2601.10967 (UP Diliman) | Multi-objective mathematical optimization of Wolbachia release timing. Uses Philippine data. Not RL. **Closest Philippine prior art.** |
| Dengue forecasting (Philippines) | medRxiv 2020; PMC3983113; PMC5905126; PMC12543447 | Supervised learning — predicts case counts. Does not decide what interventions to take. |

### 8.4 The gap table

| Combination | Status in literature | Thesis relevance |
|---|---|---|
| Dengue + any RL | **Empty — zero papers** | Core gap. The entire project. |
| Vector-borne SEIR-SEI + deep RL (PPO/SAC) | Empty — malaria RL used simplified bandits | The simulator design fills this. |
| Vector-control actions (fogging, LSR, larviciding) + RL | Empty — malaria RL only covers ITN/IRS | The action space design fills this. |
| Hard budget constraint (CMDP) + dengue | Empty — dengue budget work is non-RL | The CMDP formulation fills this. |
| Partial observability + epidemic RL | Empty — all epidemic RL is fully observable | The POMDP extension fills this. |
| Delayed reward / credit assignment + epidemic RL | Empty — all epidemic RL uses dense rewards | The credit assignment experiments fill this. |
| Philippine dengue + RL | Empty — PH work is ABM / forecasting / optimization | The Philippine calibration fills this. |

---

## 09 Open research questions

These are the questions the thesis will attempt to answer. Ordered from most foundational to most exploratory. The first two are required for any version of the thesis. The rest can be scoped in or out.

### Q1 — Can a constrained RL agent outperform hand-crafted baselines on a dengue SEIR-SEI simulator? *(Required)*

This is the baseline question. If PPO with Lagrangian budget constraints does not beat a simple rule-based policy (e.g., "fog everywhere when cases exceed threshold, stop when budget is exhausted"), the project has a problem. This must be established first.

*Baseline to beat: Mai et al.'s approach applied to a dengue environment — PPO with cost-in-reward, dense reward, full observability.*

### Q2 — How does delayed reward affect learning, and which credit assignment method best addresses it? *(Required)*

This is the core methodological question. Compare: standard PPO (discounting only) vs PPO with reward redistribution vs PPO with HER vs PPO with a learned world model. Measure: convergence speed, final policy quality, and variance across seeds.

### Q3 — How much does partial observability (underreporting) degrade policy performance?

Train a policy under full observability. Evaluate it under partial observability. Measure the performance gap. Then train a recurrent policy directly under partial observability. Is the gap recoverable?

### Q4 — How robust is the trained policy to ODE parameter misspecification?

Train on a calibrated ODE. Evaluate on ODE variants with perturbed parameters (e.g., transmission rate ±30%, mosquito lifespan ±50%). Tests whether the policy has learned generalizable behavior or overfit to the specific simulator.

### Q5 — Can the policy's recommendations be explained in terms a health officer finds actionable?

Qualitative: show the explanation outputs to domain experts (ideally actual health officers or DOH staff) and evaluate whether they are trusted and acted upon. Quantitative: measure whether SHAP-based explanations correctly identify the features that drive the policy's decisions.

---

## 10 Reading list

Priority-ordered. Read in sequence. Each item includes a one-line reason for why it is there.

### 10.1 Foundations (read first)

| Resource | Why |
|---|---|
| Sutton & Barto, *Reinforcement Learning: An Introduction*, Chapters 1–6 | The canonical textbook. Chapters 1–3 give you MDP and Bellman equations intuitively. Chapters 4–6 cover temporal difference learning and Q-learning. Free online. |
| OpenAI Spinning Up in Deep RL (online) | The most accessible practical introduction to deep RL. Builds from policy gradients to PPO. Includes code. Read alongside S&B. |
| Gymnasium documentation (gymnasium.farama.org) | The standard RL environment interface. You will implement your simulator to this standard. |
| Keeling & Rohani, *Modeling Infectious Diseases*, Chapters 1–3 (free online) | The epidemiology foundations. Chapters 1–2 give you SIR and SEIR models intuitively. Chapter 3 covers vector-borne diseases including the SEI mosquito model. |

### 10.2 Prior work (read before building anything)

| Paper | Why |
|---|---|
| Mai et al., IJCAI 2023 (arXiv:2301.12802) | The backbone you extend. Understand every design choice they made before you diverge from it. |
| Miksch et al. 2016/2019 — Cebu City dengue ABM | The only Philippine dengue simulation. Your ODE parameters should be consistent with their ABM calibration. |
| Corum et al., arXiv:2601.10967 — Wolbachia optimization, UP Diliman | The closest Philippine dengue intervention-optimization prior art. Know what they did so you can position your work relative to it. |
| Peng & Perrault, arXiv:2603.19397 — Constrained RMAB for outbreak control | The best CMDP-style public health RL method. Your Lagrangian PPO implementation should be informed by their approach. |
| Altman, *Constrained Markov Decision Processes*, 1999 | The theoretical foundation for CMDP. Focus on chapters 1–3 for the Lagrangian formulation. |

### 10.3 Credit assignment methods (read when designing experiments)

| Paper | Why |
|---|---|
| Andrychowicz et al. 2017 — Hindsight Experience Replay (HER) | Core credit assignment method. Short, clear, highly cited. Understand before implementing. |
| Arjona-Medina et al. 2019 — RUDDER: Return Decomposition for Delayed Rewards | The primary reward redistribution method. Directly addresses long-horizon credit assignment. |
| Hafner et al. 2023 — DreamerV3 | World model approach. Trains an agent to plan in imagination. Relevant if the ODE simulator is too slow for direct RL training. |

### 10.4 Data sources

| Source | What it provides |
|---|---|
| DOH Philippines PIDSR | Weekly dengue case counts by region and province. Available via DOH epidemiology bureau reports and HDX. |
| PAGASA | Historical and forecast rainfall data. Weekly and monthly rainfall by station. Essential for seasonal forcing calibration. |
| Philippine Statistics Authority (PSA) 2020 Census | Population denominators by barangay, municipality, and province. Required for incidence rate calculation. |
| HDX — Humanitarian Data Exchange (data.humdata.org) | Aggregated Philippine dengue datasets. Search "Philippines dengue". |
| PhilGIS / NAMRIA | Philippine administrative boundary shapefiles. Required for any spatial structure. |

---

## 11 Glossary

Every technical term used in this document, defined in plain language. Terms are also defined inline on first use — this section is the quick lookup.

| Term | Plain language definition |
|---|---|
| **Action space** | The complete set of actions available to the RL agent at each timestep. Can be discrete (a fixed list of options) or continuous (a range of values). |
| **Agent** | The RL learner. The entity that observes the environment, takes actions, and updates its policy based on reward. |
| **Bellman equation** | A recursive equation expressing the value of a state as the immediate reward plus the discounted value of the next state. The foundational equation of RL. |
| **Budget constraint** | A hard limit on the total resources the agent can spend. In the dengue problem, the monthly or seasonal health intervention budget in pesos. |
| **CMDP** | Constrained Markov Decision Process. An MDP with one or more hard constraints on cumulative cost. |
| **Compartmental model** | An epidemiological model that divides a population into discrete disease states and tracks flows between them. SEIR and SEIR-SEI are compartmental models. |
| **CPO** | Constrained Policy Optimization. An RL algorithm that guarantees constraint satisfaction during training. More rigorous than Lagrangian methods but harder to implement. |
| **Credit assignment problem** | The challenge of determining which past action in a trajectory deserves credit for a delayed reward. |
| **Dense reward** | A reward received at every timestep. Contrast with sparse reward. |
| **Discount factor (γ)** | A number between 0 and 1 that weights future rewards. γ = 0.99 means a reward 100 steps away is worth about 37% of a reward now. |
| **Environment** | In RL, the world the agent interacts with. Receives actions, returns observations and rewards. |
| **Extrinsic incubation period** | The time between a mosquito biting an infectious human and the mosquito becoming infectious. Approximately 8–12 days for dengue. |
| **Gymnasium** | The standard Python interface for RL environments (`reset`, `step`, `observation_space`, `action_space`). Maintained by Farama Foundation. |
| **HER** | Hindsight Experience Replay. A credit assignment method that relabels failed trajectory episodes as successes for alternative goals. |
| **Intrinsic incubation period** | The time between a human being infected and becoming infectious. Approximately 5–7 days for dengue. |
| **Lagrangian relaxation** | A method for handling constraints by adding a penalty term (multiplier × constraint violation) to the objective function. |
| **LSR** | Larval Source Reduction. A dengue intervention that removes standing water where *Aedes* mosquitoes breed. |
| **MDP** | Markov Decision Process. The mathematical framework for sequential decision-making under uncertainty. |
| **MLP** | Multi-Layer Perceptron. A standard feedforward neural network. Default policy architecture for vector state spaces. |
| **ODE** | Ordinary Differential Equation. A differential equation in which the unknown is a function of time. SEIR models are systems of ODEs. |
| **Off-policy** | An RL setting where the agent can learn from experience generated by a different policy (stored in a replay buffer). SAC is off-policy. |
| **On-policy** | An RL setting where the agent can only learn from experience generated by its current policy. PPO is on-policy. |
| **Partial observability** | A setting where the agent cannot observe the true state directly — only a noisy or incomplete observation. |
| **PIDSR** | Philippine Integrated Disease Surveillance and Response. The DOH surveillance system that collects weekly dengue case reports. |
| **Policy** | A function mapping states to actions (or distributions over actions). The output of RL training; what is deployed in production. |
| **POMDP** | Partially Observable MDP. An MDP where the agent receives noisy observations rather than the true state. |
| **PPO** | Proximal Policy Optimization. A policy gradient RL algorithm with stable, clipped updates. The recommended starting algorithm for this project. |
| **Reward hacking** | When an agent maximizes the reward signal in a way that does not correspond to the intended behavior. A risk when using proxy rewards. |
| **Reward shaping** | Adding intermediate rewards to help an agent learn faster. Risks introducing bias toward the proxy signal. |
| **RUDDER** | Return Decomposition for Delayed Rewards. A credit assignment method that redistributes delayed reward back to the actions that caused it. |
| **SAC** | Soft Actor-Critic. An off-policy RL algorithm that maximizes both reward and policy entropy. More sample-efficient than PPO but harder to tune. |
| **SEIR** | Susceptible → Exposed → Infectious → Recovered. Standard compartmental model for diseases with an incubation period. |
| **SEIR-SEI** | The host-vector dengue model: SEIR for humans + SEI for mosquitoes, coupled through the biting rate. |
| **SEI** | Susceptible → Exposed → Infectious. The mosquito compartmental model. No recovery compartment — mosquitoes stay infectious until death. |
| **Serotype** | A variant of a pathogen distinguished by surface antigens. Dengue has four (DENV-1 to DENV-4). Recovery from one does not confer full immunity to others. |
| **SHAP** | SHapley Additive exPlanations. A method for attributing the contribution of each input feature to a model's output. Used for the explanation module. |
| **Sim-to-real gap** | The performance degradation when a policy trained in simulation is deployed in the real world, due to differences between simulator and reality. |
| **Sparse reward** | A reward received only rarely (e.g., at episode end). Makes credit assignment harder. |
| **State space** | The set of all possible states the environment can be in. Includes case counts, rainfall, budget, week, and intervention history in the dengue MDP. |
| **Timestep** | A single discrete unit of time in the MDP. One call to `step()` in the environment. Tentatively one week in this project. |
| **Value function** | A function estimating expected cumulative discounted reward from a given state V(s) or state-action pair Q(s,a). The core object RL algorithms learn. |
| **Vector** | In epidemiology: an organism (here *Aedes aegypti*) that transmits a pathogen between hosts. Not to be confused with the mathematical vector. |
| **Wolbachia** | A bacterium introduced into *Aedes* mosquitoes that suppresses dengue transmission. Being trialed in several dengue-endemic countries including the Philippines. |

---

*End of AEGIS.md · This document grows with the project. When something is decided, update it. When something new is discovered, add it. When the north star shifts, rewrite section 01 first.*
