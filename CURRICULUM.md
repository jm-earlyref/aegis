# Aegis — From Zero RL to a Working Baseline

**A 4-week, build-first curriculum. Target: answer Research Question Q1 by July.**

> This is the **engineering-PoC track** — the watered-down regime, built to produce one figure. The **research track** that picks up after it (re-hardening the full problem and making the thesis contribution) lives in `THESIS.md`. Read that for *where this is going*; read this for *how to start moving*.

---

## Read this first — the whole climb on one page

You are not entering a black box. Here is the entire logic of the next four weeks, so you can always see where you are and why.

**The north star (never lose this):** a barangay health officer, Monday morning, ₱50,000 left, incomplete case reports, deciding where to fog. You are building the agent that tells them where to act and when — and earns their trust. Everything below is in service of that one image.

**The July target is narrower than the north star — on purpose.** You don't build the whole tool by July. You build the first piece of *evidence* that the tool could ever work: **Q1 — a constrained RL agent that beats a fixed-schedule baseline on a dengue simulator, within budget.** That single result is what makes the thesis real. The rest of the system is built on top of it later.

> This curriculum is only the engineering climb to the Q1 figure (the watered-down regime). The full problem — and the research contribution built on top of this PoC — is framed in `THESIS.md`. The thesis stakes **one** axis (partial observability) as its scientific claim; this curriculum builds the baseline that claim is measured against.

**Right now, four things stand between you and that result. Each phase removes exactly one:**

| You currently can't… | …because you've never | The phase that fixes it | After it, you can |
|---|---|---|---|
| Picture what "an agent learning" even *is* | watched one train | **Phase 0** | read every RL concept as a concrete thing, not an abstraction |
| Build or debug RL, only describe it | written `reset`/`step` or trained anything | **Phase 1** | treat any environment as "the same skeleton with harder physics inside" |
| Give the agent a world to act in | turned dengue dynamics into code | **Phase 2** | run a controllable epidemic — the officer's reality, compressed |
| Prove the agent helps anyone | trained a policy against a real baseline | **Phase 3** | show the Q1 figure: evidence the tool could earn trust |

Read that table when you feel lost. It is the map. Every step below tells you which incapacity it's chipping away and what it unlocks next.

> **The rule:** every step ends in a tangible artifact — a script that runs, a plot you can show, a paragraph you can explain. No artifact means you've drifted into theory you don't need yet. Come back.

> **The trap (your own notes flagged it):** trying to build SEIR-SEI + Lagrangian PPO + HER + SHAP + web UI at once. You won't. This curriculum builds *one* thing and protects it ruthlessly. A small thing that runs beats a big thing that doesn't, every single time.

---

## The shape of the four weeks

| Phase | Days | You build | The incapacity it removes |
|---|---|---|---|
| **0 — The loop in your hands** | 1–3 | PPO on CartPole | "I can't picture an agent learning" |
| **1 — Concepts via code** | 4–9 | A tiny custom Gym env + real PPO understanding | "I can describe RL but can't build it" |
| **2 — The dengue simulator** | 10–18 | SEIR-SEI ODE as a Gymnasium env | "The agent has no world to act in" |
| **3 — Train & constrain** | 19–28 | PPO beats a baseline under budget | "I have no proof any of this helps" |

If you're behind at the end of a phase, cut scope *inside the next phase* — never skip the artifact.

---

## Phase 0 — The RL loop in your hands (Days 1–3)

**The incapacity:** You understand RL as a *problem class* but you've never seen an agent learn. So right now, every term in AEGIS.md — value function, policy, advantage — is a word, not a thing. We fix that before reading another sentence.

**Why this comes first and not the theory:** Reading about RL before seeing it is like reading about swimming on dry land. Three days from now, when S&B says "the value function estimates expected future reward," you'll think "oh — the thing that made my CartPole reward curve climb." That's the difference between memorizing and understanding.

### Step 0.1 — Environment setup
```bash
pip install gymnasium stable-baselines3[extra]
```
**Why:** These two libraries are the exact tools you'll train your dengue agent with in Phase 3. You're installing your real toolkit now, on an easy problem, so the tooling is never the thing that breaks later.
**Artifact:** `import gymnasium, stable_baselines3` runs clean.

### Step 0.2 — Train CartPole and watch it
```python
import gymnasium as gym
from stable_baselines3 import PPO

env = gym.make("CartPole-v1")
model = PPO("MlpPolicy", env, verbose=1)
model.learn(total_timesteps=50_000)
```
Then render a trained episode.

**Why:** PPO is the *exact algorithm* the literature (Mai et al.) uses and the one you'll extend. CartPole is just a safe place to meet it. When you watch the pole go from flailing to balanced, you've watched the precise mechanism that will one day learn to sequence fogging and larviciding. Same algorithm. Harder world.
**Source:** Stable-Baselines3 "Getting Started" — `stable-baselines3.readthedocs.io`
**Artifact:** A clip of the pole balancing, and you can say out loud what the observation, action, reward, and episode were. If you can't name them, you're not done — those four words are the spine of everything.

### Step 0.3 — Name the parts, on the easy problem
Map CartPole onto the MDP table from AEGIS.md §3.1 (State / Action / Reward / Transition / γ), one sentence each.

**Why this is sneaky-important:** This is the *exact table you will fill in for dengue* in Phase 2. You're practicing the translation move — "real situation → MDP components" — on a problem where it's obvious, so that when the situation is a health officer's Monday morning, the translation is a habit, not a struggle.
**Artifact:** A 5-line note. Keep it. You'll write its dengue twin in two weeks.

---

## Phase 1 — Core concepts, learned through code (Days 4–9)

**The incapacity:** You can explain *why* RL fits dengue, but you can't build it, debug it, or read an algorithm section of a paper. Your understanding map names this exactly: you know RL as a problem, not as an implementation. This phase closes that gap — and not one inch more, because you don't need more for July.

**Why we don't do a full RL course:** A course teaches you all of RL. You need the ~20% that Aegis runs on: MDPs, value, policy gradients, and PPO specifically. Learning the other 80% now is time stolen from your simulator. We come back for more only when a real problem demands it.

### Step 1.1 — The minimum theory, read for intuition
- **Sutton & Barto, Ch. 1–3** (free: `incompleteideas.net/book/the-book.html`). Ch.3 = MDPs, value, Bellman — the spine.
- **OpenAI Spinning Up, Parts 1–2** (`spinningup.openai.com`).

**Why these chapters and not the book:** Ch.3 gives you the Bellman equation, which is the machinery behind *why delayed reward is hard* — and delayed reward (cases drop weeks after you fog) is one of your three core constraints and a core research thread (a deferred axis in `THESIS.md`).
**Artifact:** In your own words, 3–4 sentences: *Why does a delayed reward make the value function hard to estimate?* When you can answer this, you've understood the seed of one of the thesis's research threads.

### Step 1.2 — Understand PPO specifically (hands-on)
- **Spinning Up → PPO** page (intuition + pseudocode).
- **CleanRL's single-file `ppo.py`** (`docs.cleanrl.dev`) — read top to bottom, ~300 complete lines.
- Optional, hands-on: **Hugging Face Deep RL Course**, PPO unit (`huggingface.co/learn/deep-rl-course`).

**Why PPO and only PPO:** It's the algorithm Mai et al. used, which means using it makes your work *directly comparable to the closest prior art* — a free win for your related-work section. It also handles both discrete and continuous actions, so you won't have to relearn an algorithm when you revisit your action-space decision. Master one tool deeply rather than five shallowly.
**Artifact:** Explain what PPO's clip does and why it stabilizes training, in 3 sentences. This is the first RL algorithm you'll *understand* rather than cite — and it's the one your thesis runs on.

### Step 1.3 — Build a tiny custom Gym environment (the secret keystone)
Build something deliberately dumb: a thermostat (state = temp, actions = heat/nothing/cool, reward = −distance from target). ~40 lines.

**Why this is the most important step in Phase 1 — read this carefully:** Right now the dengue simulator feels enormous and scary. After this step, it won't. Because the dengue simulator is *the exact same object* as your thermostat — a class with `reset`, `step`, `observation_space`, `action_space` — just with SEIR-SEI equations inside `step()` instead of a temperature update. By writing the easy one first, you reduce the simulator from "a terrifying unknown" to "a thing I've already built once, with harder physics." That reframe is worth more than any reading.
**Source:** Gymnasium "Create a custom environment" — `gymnasium.farama.org`
**Artifact:** SB3's PPO trains on *your* env and the reward curve climbs. The moment this works, you have personally closed the gap between "I understand RL" and "I can build RL." That's the whole point of the phase.

> **Quiet bonus:** this thermostat skeleton is the *same object* you'll reuse twice more — once for the dengue env in Phase 2, and once for the tiny toy POMDP that runs the thesis's first experiment (the existence check; see the bridge section at the end). You're building a reusable muscle, not a throwaway.

---

## Phase 2 — Build the dengue simulator (Days 10–18)

**The incapacity:** You have an agent (PPO) and you know how to wrap a world (Gym env), but the agent has no dengue world to act in. This phase builds that world. This is where the *domain becomes the contribution* — nobody has built a vector-borne SEIR-SEI RL environment with real interventions. You are making the thing that doesn't exist in the literature.

**Why this gets the most time:** Your understanding map calls simulator calibration the hardest part of the project, and it's right. But notice *why* it's hard — it's epidemiology and parameter-fitting, not RL. Thanks to Phase 1, the RL plumbing is already a solved problem for you. The only new difficulty here is making the disease behave realistically. That's a much smaller mountain than it looked like in week one.

### Decision shortcut — kill analysis paralysis now
Your map's red flag #3 is you freezing on the ⚠ OPEN questions. Don't. These are already decided for the PoC (`DECISIONS.md`). Use them and move. **Every one is reversible in the research phase — choosing fast and wrong costs you a day; choosing slow and "right" costs you the deadline.**

| Open question | Pick for v1 | Why this is the safe simplification |
|---|---|---|
| Geographic scope | Single municipality | No spatial/multi-agent complexity — that's a whole separate paper |
| Timestep | 1 week | Matches DOH cadence *and* shortens your horizon |
| Season length | 26 weeks | Shorter horizon = easier credit assignment = faster training |
| Action space | Discrete | Trains far more reliably; PPO handles it; continuous is a v2 upgrade |
| Interventions | Fog + larvicide + LSR | The three with clearest model mappings; vaccines/Wolbachia are v2 |
| Observability | Full (no noise) | Partial obs is the thesis claim — you *need* a clean baseline before measuring its cost |
| Stochasticity | Deterministic | Reproducible and fast; you validate on stochastic later |
| Budget | Single scalar | Line-items are a realism upgrade, not a baseline requirement |

**Why "full observability" when partial obs is your whole thesis:** Because you can't measure how much underreporting *hurts* until you have a clean full-observability number to compare against. The full-observability agent you build here is literally the **ceiling** in your thesis's three-agent comparison (`THESIS.md` §08: full-obs ceiling / strong recurrent memory / inference). The baseline isn't a compromise — it's the control group for your most interesting experiment. You're not skipping the hard part; you're setting up to measure it.

### Step 2.1 — Epidemiology foundations
- **Keeling & Rohani, Ch. 1–3** (Ch.3 = vector-borne / host-vector models).
- Re-read AEGIS.md §2.2–2.3 with the textbook open.

**Why now and not earlier:** You read these *after* you can already build a Gym env, so you can read them asking one focused question: "how do I turn this into a `step()` function?" — instead of reading them as abstract biology. The textbook becomes a spec for code.
**Artifact:** A labeled SEIR-SEI diagram with every flow arrow tagged with its rate parameter. This is your blueprint for the next step.

### Step 2.2 — Implement the ODE (no RL yet)
Write the equations as a Python function; integrate with `scipy.integrate.solve_ivp`. Add seasonal forcing from a weekly rainfall series. Parameter starting points: Miksch et al. (Cebu City) — track every parameter's source in a spreadsheet.

**Why ODE-first, RL-never-yet:** You isolate the hard new thing (the disease dynamics) from the thing you already trust (RL). If the epidemic curve looks wrong, you *know* it's the biology, not the agent — because there's no agent yet. Debugging one unknown at a time is how you stay sane.
**Why the parameter spreadsheet matters beyond coding:** Your edge in this whole project is operational honesty. A table showing exactly which parameters came from literature vs data vs judgment *is* that honesty made concrete — and it's a ready-made chunk of your methods section. (It's also your honest answer to the fact that calibration to Dasmariñas is ill-posed: you're *documenting uncertainty*, not pretending to a precise fit. See `AEGIS.md` §7.2.)
**Artifact:** A no-intervention 26-week plot that *looks like a real dengue season* — rises with the rains, falls after. If it's flat or explodes, debug here, in pure biology, before RL ever enters.

### Step 2.3 — Add interventions as parameter perturbations
Each intervention modifies an ODE parameter for a duration (AEGIS.md §2.3). Fogging drops I_m sharply but briefly; larviciding cuts recruitment over ~2 weeks; LSR lowers carrying capacity persistently.

**Why this step is quietly thrilling:** When you overlay "no intervention" vs "fog weekly" and watch the rebound after fogging stops, you are *seeing your own combo insight* render itself in code. The reason no single move works, which you can already explain to a stranger, is now visible on a plot. The agent's job (sequencing these) is suddenly obviously hard and obviously learnable.
**Artifact:** Two diverging curves showing an intervention working, rebound included.

### Step 2.4 — Wrap it as a Gymnasium env
Reuse the Phase 1.3 skeleton. `observation` = reported cases + week + rainfall + budget + recent interventions. `action` = small discrete menu. `reward` = cases_averted − cost_weight × cost. `step()` = apply action → integrate one week → return.

**Why this feels easy now (and would have been terrifying in week 1):** This is the moment Phase 1.3 pays off. You are not learning the Gym interface and the epidemiology simultaneously — you learned the interface on a thermostat, and the biology in 2.1–2.3. Now you just snap them together. The scariest-sounding step in the project is the one you're most prepared for.
**Artifact:** `reset()`/`step()` work; a random-action 26-week rollout completes and returns sensible cumulative reward. You now have a dengue world an agent can live in.

> **This is contribution #1.** The environment you just built is the open, RL-ready dengue POMDP the thesis ships as a community artifact (`THESIS.md` §04) — the instrument that doesn't exist for dengue the way OpenMalaria does for malaria. You build it once, here, for the PoC; you harden it later (documentation, the observation-structure knob, baselines) into the released version.

---

## Phase 3 — Train and constrain (Days 19–28)

**The incapacity:** You have an agent and a world, but no *proof* the agent helps the health officer at all. This phase produces that proof. Everything so far has been setup for one figure.

**Why this answers Q1 and why Q1 matters:** Q1 asks whether a learning agent can beat the fixed schedules health officers use today. If yes, the entire premise of Aegis holds — there's something here worth a health officer's trust. If no, you've learned that early and cheaply. Either way, this is the result your defense stands on. Note what Q1 is *not*: it is not the thesis's scientific contribution (that's the observability characterization in `THESIS.md`). Q1 proves the *premise*; the contribution comes after.

### Step 3.1 — Build the baselines to beat
Code two fixed policies: do-nothing (counterfactual), and a threshold rule ("fog when cases exceed X until budget runs out" — roughly what officers do now).

**Why baselines come before the fancy agent:** "My agent works" is meaningless without "compared to what." The threshold rule *is the status quo* — the thing in the health officer's hands today. Beating it isn't an academic benchmark; it's the literal claim that your tool is better than what they currently do. That's the north star, made measurable.
**Artifact:** A table of cumulative cases + spend for both baselines.

### Step 3.2 — Train vanilla PPO (no hard constraint yet)
SB3 PPO, full observability, cost folded into reward — exactly the Mai et al. setup.

**Why replicate Mai et al.'s setup first:** It's your honest starting line. By matching the closest prior work before you improve on it, every later gain is cleanly attributable to *your* additions. You're building the control group for your own work.
**Why the frustration here is the curriculum, not a detour:** Reward scaling, episode length, learning rate will fight you. Your understanding map *predicted* this is where real understanding of PPO lives — not in the reading, in the debugging. When you fix a reward curve that won't climb, you'll finally own the algorithm.
**Source:** Mai et al., IJCAI 2023 (arXiv:2301.12802); SB3 PPO docs.
**Artifact:** A climbing reward curve and an agent that beats the threshold baseline. If it doesn't beat it, that's a finding — debug reward shaping before moving on.

### Step 3.3 — Add the hard budget constraint (the CMDP — the Q1 milestone)
Make budget a *constraint*, not a penalty (AEGIS.md §3.3). Start with a naive big penalty, watch it overspend anyway — *that failure is the lesson* about why the field needed CMDPs at all. Then use a Lagrangian approach via `sb3-contrib` or **OmniSafe** rather than hand-rolling the λ update.

**Why you let the naive version fail on purpose:** You'll truly get Lagrangian methods only when you watch the budget break and have to fix it. So we *engineer that moment*. The "why can't I just penalize cost?" frustration is the door into understanding the entire CMDP literature — Altman 1999, the whole §3.3. You learn the theory because the problem forced you to.

**Honest reframe — what this step is and isn't.** The hard constraint is what makes your PoC respect the officer's actual world (she cannot overspend ₱50,000, ever), and beating the baseline *while staying within budget* is the Q1 result that makes the premise real. But be clear-eyed: the budget constraint is **not** the thesis's research contribution. On an observable, deterministic *spend* budget, the hard limit is largely satisfiable by masking, so the Lagrangian work here is a genuine learning milestone more than a novel result (`AEGIS.md` §3.3 flag). The thesis's actual contribution is the **observability** characterization (`THESIS.md`). So: build this, learn from it, get the figure — but don't mistake the CMDP for the science.

**Artifact (the Q1 result):** One plot — learned policy vs both baselines — showing **more cases averted while respecting the budget.** That figure is the spine of your defense. When you have it, update `AEGIS.md` §09: Q1, answered, evidence attached. The premise is now real.

---

## What is OUT of scope until the Q1 figure exists

Writing these down so you can *stop carrying them.* Each connects to the north star — and each is built *on top of* a working baseline, never instead of one. All of these are now framed as the research challenge in `THESIS.md`:

- **Partial observability / POMDP — now the thesis's *claimed* contribution axis (`THESIS.md`):** the officer only sees ~25% of cases, and the underreporting *worsens during outbreaks* (state-correlated). You measure that cost *after* you have the full-obs number to compare against. The baseline you build in Phase 3 is the prerequisite, not a rival. The method is held open — world model is the leading candidate, not a commitment.
- **Credit assignment (RUDDER / HER / world models):** a *deferred* research axis (`THESIS.md` §10), not the headline. The short 26-week horizon may make it mild; you'd pick a method only if the research phase reaches it and the baseline visibly struggles.
- **Explainability (SHAP/attention):** the officer must understand *why* — but the agent has to work before it can explain itself. v2.
- **The web tool:** a thin wrapper around a trained policy. The last 5%, not the first.
- **Continuous actions, multi-category budget, province scale:** realism upgrades, each reversible, each a v2 refinement.

None of these are gaps you're ducking. They're the *next* contributions — now framed as the research challenge in `THESIS.md` — and they all stand on the figure you build by July.

---

## The bridge to the thesis (the day after the Q1 figure)

The Q1 figure ends the PoC and opens the research phase. You do **not** start the full POMDP machinery next. You start with the cheapest possible question (`THESIS.md` §08): *does the effect even exist?*

**The existence experiment.** Before the dengue partial-obs work, build a tiny toy POMDP — the *same* `reset`/`step` skeleton from Phase 1.3, with a controllable observation model. Two conditions at **matched magnitude**: corruption that depends on the hidden state (state-correlated) vs. corruption that doesn't (state-independent). Two agents: a tuned recurrent (memory) baseline and a full-observability ceiling. One plot. The gate: does the memory agent's gap-to-ceiling blow up *specifically* in the state-correlated condition? If yes, the thesis has a real phenomenon and you proceed. If no, you've learned it cheaply and reframe — before sinking months into a world model.

That toy is also the abstract end of your generality evidence. It is the natural first artifact of the research phase, and it reuses the muscle you built in Phase 1.3 — which is why none of this curriculum is throwaway.

---

## The one-glance source list

**RL fundamentals**
- Sutton & Barto (free): `incompleteideas.net/book/the-book.html`
- OpenAI Spinning Up: `spinningup.openai.com`
- Hugging Face Deep RL Course: `huggingface.co/learn/deep-rl-course`

**Code you'll actually use**
- Gymnasium: `gymnasium.farama.org`
- Stable-Baselines3: `stable-baselines3.readthedocs.io`
- CleanRL (read single-file PPO): `docs.cleanrl.dev`
- sb3-contrib / OmniSafe (constrained RL) — verify current docs

**Domain + prior work**
- Keeling & Rohani, *Modeling Infectious Diseases*, Ch. 1–3
- Mai et al., IJCAI 2023 — arXiv:2301.12802
- Miksch et al. 2016/2019 (Cebu City dengue ABM) — parameter source
- SciPy `solve_ivp`: `docs.scipy.org`

> Research-phase sources (the strong recurrent baseline, the inference-vs-memory anchors, MNAR scaffolding) live in `AEGIS.md` §11.3 and `THESIS.md` §07 — you don't need them for the PoC. Links and library APIs drift; if an address has changed, search for the current version.

---

## The weekly reality check

Every Friday ask: *Can I show my supervisor something that runs?*
- Week 1 → CartPole + your thermostat env. ("I can build RL now.")
- Week 2 → a dengue epidemic curve you can perturb. ("I built the world.")
- Week 3 → PPO training on the dengue env. ("The agent lives in it.")
- Week 4 → the Q1 figure. ("It beats the status quo.")

If a week slips, the fix is never "study more theory." It's "shrink the artifact until it runs, then grow it."

---

## When it feels too big — read this

Level 1 — *understanding the problem* — is already done. That's the part most people never finish, and you've finished it: you can explain the SEIR-SEI loop, the combo problem, the constraints, the gap in the literature, and now the precise scientific question (state-correlation; memory vs inference). What's left is reading and building, and you've scoped it to a single figure four weeks out.

You are not climbing the whole mountain. You drew the map of the mountain already. Now you're walking to one clearly-marked viewpoint — the Q1 figure — and from there you'll be able to see the rest of the trail. One step at a time, each one connected to the health officer on that Monday morning. That's the whole job.