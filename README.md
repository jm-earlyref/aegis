# Aegis

**Reinforcement learning for dengue fever intervention sequencing in the Philippines.**

> *Can we build an RL agent that learns intervention policies good enough to be trusted by a real health officer, trained on the kind of incomplete, noisy, resource-constrained reality they actually live in?*

---

## What this is

Aegis is a research project and CS thesis exploring whether reinforcement learning can learn *when*, *where*, and *in what order* to deploy dengue interventions — fogging, larviciding, larval source reduction — given a fixed budget, delayed feedback, and incomplete surveillance data.

The name comes from *aegis* — a shield. And from *Aedes aegypti* — the mosquito that carries dengue. A system that learns to protect communities from the vector that threatens them.

The project runs on two tracks. An **engineering proof-of-concept** (to July) builds a working, deliberately simplified version of the system. The **thesis** then re-introduces the full difficulty of the problem and makes a focused research contribution. See `THESIS.md` for the research framing and `CURRICULUM.md` for the PoC build.

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
- The problem is formally a **constrained, partially observable Markov decision process**

Supervised learning can predict where dengue risk is high. RL decides what to do about it, in what order, with what you have.

---

## Why this is hard

The health officer's reality is hard along several axes at once — delayed feedback, incomplete (and *biased*) reporting, a hard budget, the choice of where to act, and a world that won't match any model. That full landscape is what makes the problem real. It is **not** the contribution.

A single thesis cannot conquer all of that, and claiming to would be weaker, not stronger. So the thesis **stakes one axis** as its scientific claim — **partial observability**, the fact that the agent only ever sees a noisy, delayed, ~25% shadow of the true outbreak — and frames the rest honestly as the landscape and as future work.

Within that axis sits a sharp, specific question. Dengue underreporting gets *worse during outbreaks* — the signal is most suppressed exactly when the true epidemic is largest. This is **state-correlated** observation corruption, and the thesis asks whether it is the property that determines when an agent must *infer* the hidden state rather than merely *remember* past observations. (See `THESIS.md`.)

---

## What the literature has done

The epidemic RL literature is overwhelmingly COVID/NPI-focused. The closest prior work is Mai et al. (IJCAI 2023), which formulates multi-intervention epidemic planning as an MDP over the EpiPolicy simulator using PPO — the natural backbone to extend. Vector-borne RL exists almost entirely for malaria (IBM Research-Africa, KDD Cup 2019), using simplified bandits without budget constraints or partial observability.

Two gaps matter here, and Aegis documents both honestly:

- **Domain gap (`GAP.md`):** there are zero published papers combining dengue with RL for intervention sequencing.
- **Methods gap (`GAP_METHODS.md`):** the *scientific* claim rests on a narrower, confirmed gap — no deep-RL work isolates *state-correlation of the observation process* as the property governing when inference beats memory. Clean, but narrow, and distinct from prior work on noise *magnitude* and *task type*.

---

## The two contributions

The thesis produces two things that de-risk each other:

1. **An open, RL-ready dengue environment** — a configurable SEIR-SEI partially-observable simulator (Gymnasium-compatible) with a knob for observation structure. The instrument that does not yet exist for dengue the way OpenMalaria did for malaria. Useful to the whole community on its own.
2. **A characterization of when inference beats memory** — the empirical study tying the inference-vs-memory advantage to the *structure* of observation corruption.

If the method underwhelms, the environment still stands. If the environment is modest, the method still stands.

---

## The target output (product)

A trained RL agent wrapped in a simple web tool. A health officer inputs their region, current week, and available budget. The system returns a recommended intervention calendar, projected cases averted versus a fixed-schedule baseline with a confidence range, and a plain-language explanation of each recommendation. Explainability is a prerequisite for trust, not an optional feature. Full product spec in `PRODUCT.md`.

---

## Research philosophy

This project is designed around a specific model of research: **problem-first, method-as-necessary, contribution-as-discovery.**

The inspiration is AlphaFold — not in ambition, but in approach. AlphaFold didn't ask "what's the most theoretically interesting thing we can do with deep learning?" It asked "how do we solve protein folding?" and invented whatever the problem demanded. The theory followed the problem.

Aegis applies the same logic. The health officer's reality is the constraint that forces invention. This is also why the *method* for the chosen axis is deliberately **held open** — the world model is the leading candidate, not a foregone commitment. The problem decides.

A second principle runs through the project: **novelty is found indoors, in the literature; significance is discovered outdoors, by experiment.** A novel idea proves nothing until the world responds to it. So the research plan resolves novelty first, then runs the cheapest possible experiment to find out whether the effect is even real — before committing months to it.

---

## Current status

**Completed:**

- [x] Problem framing and north star
- [x] Domain literature survey — dengue + RL gap confirmed (`GAP.md`)
- [x] Methods literature survey — observation-structure gap confirmed, narrow (`GAP_METHODS.md`, `research/`)
- [x] Formal MDP / CMDP / POMDP problem structure (`AEGIS.md` §03)
- [x] Product / IPO model (`PRODUCT.md`)
- [x] PoC design decisions — ODE, weekly timestep, 26-week season, discrete actions, full-observability baseline (`DECISIONS.md`)
- [x] Research framing — contribution axis chosen (partial observability), methods held open, two-contribution structure (`THESIS.md`; Decisions #14–#17)
- [x] PoC build curriculum (`CURRICULUM.md`)

**In progress / not yet started:**

- [ ] Existence experiment — does memory fail *specifically* under state-correlated corruption? (the gate)
- [ ] Baseline implementation — PPO beats a threshold-rule policy under budget (Q1 PoC)
- [ ] Open dengue environment — harden the simulator and add the observation-structure knob
- [ ] Philippine DOH data acquisition and parameter grounding
- [ ] First training run

---

## Open questions

Most early design decisions are now resolved and logged in `DECISIONS.md`. The research-phase open questions — the final research-question wording, whether to build the synthetic generality environment, and the choice of inference method — live in `THESIS.md` §11. They are deliberately held open until the existence experiment reports.

---

## Repository structure

```
aegis/
├── README.md          # Front door (this file)
├── AEGIS.md           # Umbrella: domain, formal problem, RL engine
├── THESIS.md          # The research challenge: claim, question, experiments
├── PRODUCT.md         # Product & engineering: web tool, architecture
├── CURRICULUM.md      # The PoC build plan (to July)
├── GAP.md             # Domain novelty (dengue + RL)
├── GAP_METHODS.md     # Methods novelty (observation-structure characterization)
├── DECISIONS.md       # Decision log (append-only)
└── research/
    └── DSCOPDWIBMDRPOMDP.md   # Full methods-novelty literature audit
```

---

## Knowledge management

This project is documentation-heavy on purpose. It's both a build and a research project, and at this stage **the thinking is the work**. The Markdown files form a deliberate ecosystem — each owns one concern, so they inform rather than drift from each other.

| File | Owns | Read it when |
|---|---|---|
| `README.md` | The front door — what Aegis is, why, where it stands | You're new or re-orienting |
| `AEGIS.md` | The umbrella — north star, dengue domain, formal problem, RL engine | You need the whole technical picture |
| `THESIS.md` | The research challenge — the hard problem, the claimed contribution, the question, the experiments | Anything about the thesis contribution |
| `PRODUCT.md` | The product & engineering — web tool, screens, architecture, data | You're building the tool |
| `CURRICULUM.md` | The PoC build plan (the watered-down regime, to July) | You're executing the engineering PoC |
| `GAP.md` | Domain novelty (dengue + RL) | Defending "is this a real gap" |
| `GAP_METHODS.md` | Methods novelty → full evidence in `research/` | Defending the scientific claim |
| `research/` | Raw research artifacts (e.g. the literature audit) | You need the evidence behind a claim |
| `DECISIONS.md` | The decision log — every significant call, dated, with rationale | You need to know *why* something was decided |

**Conventions:**

- `DECISIONS.md` is **append-only**, newest on top. Superseded decisions are marked (`⚠ Superseded by #N`), never deleted.
- **`✓ DECIDED` / `⚠ OPEN`** markers flag what's settled versus still open throughout the docs.
- **`THESIS.md` owns the research framing.** `AEGIS.md` points to it rather than duplicating it, so the two don't diverge as the research evolves.
- When a decision changes a document, update the document **and** log the decision in `DECISIONS.md`.

---

## Reading list

If you're coming to this cold and want to understand the space, start with `AEGIS.md` (§11 has the full annotated reading list). The essentials:

- **Mai et al., IJCAI 2023** (arXiv:2301.12802) — the closest prior work; the backbone to extend
- **Sutton & Barto, *Reinforcement Learning: An Introduction*** — RL foundations; free online
- **Keeling & Rohani, *Modeling Infectious Diseases*, Ch. 1–3** — epidemiology foundations; free online
- **Ni, Eysenbach & Salakhutdinov, ICML 2022** (arXiv:2110.05038) — the strong recurrent baseline the thesis must beat
- **Miksch et al. 2016/2019** — the only Philippine dengue ABM (Cebu City); parameter reference

---

## About

Built by a 3rd year CS undergrad from Dasmariñas, Calabarzon, Philippines.

This is both a CS thesis and a genuine attempt to build something useful — a tool that earns a health officer's trust in one of the most dengue-affected regions in the world.

---

*This repository is updated as the project develops. `AEGIS.md` is the umbrella source of truth; `THESIS.md` owns the research framing; `DECISIONS.md` records why each call was made.*