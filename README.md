# Aegis

**Reinforcement learning for dengue fever intervention sequencing in the Philippines.**

> *Can we build an RL agent that learns intervention policies good enough to be trusted by a real health officer, trained on the kind of incomplete, noisy, resource-constrained reality they actually live in?*

---

## What this is

Aegis is a research project and CS thesis exploring whether reinforcement learning can learn *when*, *where*, and *in what order* to deploy dengue interventions — fogging, larviciding, larval source reduction — given a fixed budget, delayed feedback, and incomplete surveillance data.

The name comes from *aegis* — a shield. And from *Aedes aegypti* — the mosquito that carries dengue. A system that learns to protect communities from the vector that threatens them.

This repository is currently at the **idea and research design stage.** No code yet — just the thinking that will eventually become the code.

---

## The problem

In 2024, the Philippines reported 413,960 dengue cases — the highest in ASEAN. Calabarzon, where this project was conceived, is one of the most affected regions. Almost every Filipino family has a dengue story.

Every Monday morning, a barangay health officer decides where to deploy fogging, larviciding, or community cleanup. They have last week's case reports (incomplete — only about 1 in 4 cases is reported), a rainfall forecast, and a fixed monthly budget. Whatever they decide, they won't see the result for 2–4 weeks.

Right now, that decision is made using intuition and fixed schedules. There is no tool that tells them where to act and when — and actually works.

That is the problem Aegis is trying to solve.

---

## Why reinforcement learning

Dengue intervention sequencing is a **sequential decision problem under delayed feedback**:

- Interventions today change mosquito populations and transmission *weeks later*
- You must allocate a fixed budget across a whole season
- There are no labeled "optimal policies" to learn from — supervised learning cannot help here
- The problem is formally a **constrained Markov Decision Process (CMDP)**

Supervised learning can predict where dengue risk is high. RL decides what to do about it, in what order, with what you have.

---

## Why this is hard (and why that's interesting)

Three things make this problem genuinely difficult — and none of them have been tackled together before:

**1. Delayed reward.** When cases drop three weeks after fogging, the agent can't easily tell which decision caused it. This is the credit assignment problem. Standard RL methods weren't designed for 3–4 week lags.

**2. Partial observability.** The agent never sees the true epidemic state. ~75% of dengue cases are unreported. It must act on a noisy, delayed, incomplete signal.

**3. Hard budget constraints.** Standard RL folds cost into the reward — the agent can overspend if it thinks it's worth it. A real health officer cannot. The budget is a hard limit, not a preference. This requires a proper CMDP formulation, not a reward hack.

Each of these is individually studied in the RL literature. The combination, in a vector-borne disease setting, has not been tackled. That combination is the contribution.

---

## What the literature has done

The epidemic RL literature is overwhelmingly COVID/NPI-focused — lockdowns, school closures, vaccination timing. The closest prior work is Mai et al. (IJCAI 2023), which formulates multi-intervention epidemic planning as an MDP over the EpiPolicy simulator using PPO. It is the natural backbone to extend.

Vector-borne RL exists almost entirely for malaria (IBM Research-Africa, KDD Cup 2019) — using simplified bandits and basic MDPs, not deep RL, without budget constraints or partial observability.

**There are zero published papers combining dengue with reinforcement learning for intervention sequencing.** This is a confirmed negative result from a systematic literature search conducted in June 2026. The dengue control literature uses classical optimal control (Pontryagin), agent-based simulation with hand-crafted strategies, or ML forecasting. None of these involve a learning agent making sequential decisions.

---

## The target output

A trained RL agent wrapped in a simple web tool. A health officer inputs their region, current week, and available budget. The system returns:

- A recommended intervention calendar (what to deploy, where, when) for the next N weeks
- Projected cases averted vs a fixed-schedule baseline, with a confidence range
- A plain-language explanation of why each recommendation was made

The explainability requirement is not optional. It is a prerequisite for trust. An agent that can't explain itself is an agent a health officer won't use.

---

## Research philosophy

This project was designed around a specific model of research: **problem-first, method-as-necessary, contribution-as-discovery.**

The inspiration is AlphaFold — not because this project is as ambitious, but because of the approach. AlphaFold didn't start by asking "what's the most theoretically interesting thing we can do with deep learning?" It asked "how do we solve protein folding?" and then invented whatever the problem demanded. The theory followed the problem.

Aegis applies the same logic. The health officer's reality — incomplete data, hard budget, delayed feedback, a fixed seasonal window — is the constraint that forces invention. Whatever novel RL methodology emerges from this project will emerge because the problem demanded it, not because it was selected from a menu of theoretical gaps.

The dengue domain is the vehicle. The method is the contribution. The health officer's trust is the north star.

---

## How this project came to be

This project was developed through a long research design conversation that worked through the following questions in order:

**What problem do I actually care about?** Not "which RL problem should I apply to a domain" — but what real situation in the world is broken and worth fixing. The answer was dengue in the Philippines, grounded in personal experience growing up in Calabarzon.

**What does "solved" look like?** Not a benchmark number. A health officer opening a tool, seeing where to act and when, acting on it, and it working. Fewer cases. Fewer deaths. That image is the north star.

**What does that image demand technically?** Working backward from the health officer's reality to the formal problem: a CMDP with partial observability, multiple intervention types, delayed reward, and hard budget constraints. The technical requirements emerge from the human requirements.

**What has the literature done, and what hasn't it done?** A systematic survey confirmed: epidemic RL is COVID-dominated, vector-borne RL is malaria-only and shallow, dengue + RL is empty. The gap is real.

**What RL methodology does the problem demand?** This is still being worked out. The five open theoretical threads are credit assignment, exploration under constraint, world models, constrained RL, and generalization. The thesis will determine which combination the problem actually requires.

The project is named Aegis. The Pokémon EEVEE was briefly considered as an acronym. It was not used.

---

## Current status

This repository is at the **research design stage.** The following have been completed:

- [x] Problem framing and north star definition
- [x] Literature survey (June 2026) — confirmed dengue + RL gap
- [x] Formal MDP/CMDP/POMDP problem structure designed
- [x] IPO data model for the web tool
- [x] RL engine semantic specification
- [x] Full reference document (`docs/reference.md`)

The following are in progress or not yet started:

- [ ] Simulator design decision (ODE vs ABM, timestep, season length)
- [ ] Action space design (discrete vs continuous)
- [ ] Baseline implementation (reproduce Mai et al. on a dengue environment)
- [ ] Philippine DOH/PIDSR data acquisition and calibration
- [ ] First training run

---

## Open design questions

These are explicit decisions not yet made. They are tracked here honestly rather than papered over.

| Question | Options | Status |
|---|---|---|
| Geographic scope | Single municipality vs province vs region | ⚠ Open |
| Timestep | Daily / weekly / bi-weekly | ⚠ Open |
| Season length | 26 weeks vs 52 weeks | ⚠ Open |
| Action space | Discrete vs continuous | ⚠ Open |
| Interventions in scope | Fogging + larviciding + LSR only, or include vaccines + Wolbachia | ⚠ Open |
| Budget granularity | Single scalar vs multi-category | ⚠ Open |
| Partial observability handling | Ignore / observation noise / full POMDP | ⚠ Open |
| Credit assignment method | Discounting only / RUDDER / HER / world model | ⚠ Open |
| Explanation method | SHAP / attention / counterfactual / rule extraction | ⚠ Open |

---

## Repository structure

```
aegis/
├── README.md              # This file
├── docs/
│   └── reference.md       # Full technical reference document
└── ...                    # Code will live here when the time comes
```

---

## Reading list

If you're coming to this cold and want to understand the space:

- **Mai et al., IJCAI 2023** (arXiv:2301.12802) — the closest prior work; the backbone to extend
- **Sutton & Barto, Reinforcement Learning: An Introduction** — the RL foundations textbook; free online
- **Keeling & Rohani, Modeling Infectious Diseases, Ch. 1–3** — the epidemiology foundations; free online
- **Miksch et al. 2016/2019** — the only Philippine dengue ABM (Cebu City); parameter reference
- **Corum et al., arXiv:2601.10967** — Wolbachia optimization for dengue, UP Diliman; closest Philippine prior art
- **Peng & Perrault, arXiv:2603.19397** — constrained RMAB for outbreak control; best CMDP public health RL

---

## About

Built by a 3rd year CS undergrad from Dasmariñas, Calabarzon, Philippines.

This is both a CS thesis and a genuine attempt to build something useful — a tool that earns a health officer's trust in one of the most dengue-affected regions in the world.

---

*This repository will be updated as the project develops. The reference document in `docs/` is the primary technical source of truth.*
