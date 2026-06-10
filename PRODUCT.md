# PRODUCT.md
### Aegis — Software Product Document

*What we are building, why, and how. This document closes the product and engineering unknowns for Aegis. It does not resolve open RL research questions — those live in AEGIS.md. Where the two worlds intersect, this document flags it explicitly.*

*Last updated: June 2026*

---

## How to use this document

**Narrative sections (top)** — read top to bottom to understand what Aegis is, who it's for, and what it's not. Start here if you're new to the project or need to re-ground yourself before a meeting.

**Reference sections (bottom)** — jump to any section independently. Feature list, user flows, screen inventory, data model, architecture, constraints, open decisions. These stand alone.

**Flags** — anything marked ⚠ is an explicit assumption or unresolved decision. They are tracked honestly rather than papered over.

---

## 01 Problem & north star

There is no tool that tells a barangay health officer where to act against dengue, when to act, and in what sequence — and actually works.

Every Monday morning, a municipal health officer in Dasmariñas, Cavite sits down with last week's case reports (incomplete — only about 1 in 4 dengue cases is reported), a rainfall forecast, and a fixed monthly budget. They must decide: fog this week or wait? Larvicide which areas? Spend now or conserve for the outbreak peak? Whatever they decide, they won't see the result for 2–4 weeks. Right now, that decision is made using intuition, local knowledge, and fixed seasonal schedules. In 2024, the Philippines reported 413,960 dengue cases — the highest in ASEAN — with Calabarzon among the most affected regions.

The problem is not dengue forecasting. It is not epidemiological modeling. The problem is:

> *Can we build an RL agent that learns intervention policies good enough to be trusted by a real health officer, trained on the kind of incomplete, noisy, resource-constrained reality they actually live in?*

That question is the north star. Everything in this document exists in service of it.

---

## 02 What we're building

Aegis is a **web-based decision-support tool** for barangay and municipal health officers managing dengue intervention budgets in the Philippines.

The officer opens Aegis at the start of the dengue season, enters their municipality, available budget, and weekly case data. The system — powered by a trained reinforcement learning agent — returns a recommended intervention plan: what to deploy (fogging, larviciding, or larval source reduction), when, how much budget to allocate, and which barangays to prioritize. The officer sees a choropleth map of their municipality with risk levels and recommended deployment areas, an intervention calendar for the season ahead, projected cases averted versus a fixed-schedule baseline, and a plain-language explanation of why each recommendation was made.

The officer returns every week, optionally enters new case data, and receives an updated plan. The agent does not learn from deployment — it runs a frozen trained policy against current inputs.

**Aegis is a decision-support tool, not an autonomous system.** The health officer remains in the loop at every step. The recommendation is an input to their decision, not a replacement for it.

---

## 03 Goals & non-goals

### Goals

| Goal | Phase |
|---|---|
| Health officer can generate a dengue intervention plan in under 5 minutes | PoC |
| Recommendations are actionable at the barangay level | PoC |
| System explains each recommendation in plain language the officer understands | PoC |
| Agent outperforms a fixed-schedule baseline on cases averted within budget | PoC |
| Officer can update case data weekly and receive a refreshed plan | PoC |
| System is honest about uncertainty — flags low-confidence weeks explicitly | PoC |
| Agent incorporates partial observability (underreporting) into its decision-making | v2 |
| System produces validated results across multiple seeds and historical seasons | v2 |
| Officer can export the plan as a PDF for filing and supervisor approval | v2 |
| Officer can share a URL to the plan with their supervisor | v2 |

### Non-goals

- **Not a DOH deployment.** The gap between this thesis PoC and a DOH-deployable product includes prospective clinical validation, live PIDSR data pipeline, per-municipality recalibration, human factors research with real officers, robustness certification, and an institutional regulatory pathway. None of that is in scope.
- **Not an autonomous system.** Aegis never acts on behalf of the officer. It recommends. The officer decides.
- **Not a dengue forecasting tool.** Aegis does not predict case counts. It recommends intervention sequences given a current situation.
- **Not a province or region-scale tool.** Geographic scope is a single municipality — Dasmariñas, Cavite — for the PoC. Province and region scale require hierarchical RL, a separate research problem.
- **Not spatially-aware at the RL level.** The RL agent operates on municipality-level aggregates. It has no knowledge of individual barangays. Barangay-level recommendations are produced by a spatial heuristic layer on top of the agent (see Screen Inventory §07 and Architecture §09). This is explicit in the product UI and must never be misrepresented as agent intelligence.
- **Not a live surveillance system.** Aegis does not connect to PIDSR or pull live DOH data automatically. Case data is entered manually by the officer.

---

## 04 Users & context

### The health officer

**Name:** Maria (representative persona)
**Role:** Municipal Health Officer, Dasmariñas Rural Health Unit, Cavite
**Device:** Desktop or laptop at the RHU office. Occasionally a tablet. Mobile is secondary.
**Technical comfort:** Comfortable with basic web tools and government reporting systems. Not a data scientist. Will not read a chart with confidence intervals without a plain-language label.
**When she opens Aegis:** Monday mornings, before the week's field operations are dispatched. Session is under 10 minutes.
**What she has:** A logbook of cases reported to her health center last week, a sense of which barangays have been problematic this season, and a running budget tally.
**What she doesn't have:** Complete case data (75% of dengue goes unreported), certainty about whether last week's fogging worked, granular PIDSR access, or time to interpret a complex model output.
**Her frustration:** She follows the DOH 4S/5S seasonal schedule because it's what she has. She knows it's reactive — she's often fogging during a peak, not before it. She wants to act earlier, but doesn't know where to look or how to justify the decision to her supervisor.
**What earns her trust:** A tool that tells her *why* it's recommending something, in language she can repeat to her supervisor. A recommendation she can act on today — a specific barangay, a specific intervention — not a general advisory.

---

## 05 Feature list

### PoC (July)

| Feature | Description |
|---|---|
| Municipality selector | Pre-loaded with Dasmariñas, Cavite for PoC. Selects the geographic context for the plan. |
| Season week selector | Officer sets the current week of the dengue season (1–26). |
| Budget input | Total remaining budget for the season in Philippine pesos. Single scalar. |
| Barangay case data entry | Officer enters weekly reported case counts per barangay. Pre-populated with prior week values, officer updates what changed. |
| Rainfall display + override | System auto-fetches 7-day forecast from Open-Meteo for Dasmariñas coordinates. Displayed visibly. Officer can override. |
| Intervention calendar | Week-by-week recommendation table: intervention type (fogging / larviciding / LSR), target barangay, estimated cost, budget remaining. Current week highlighted. |
| Choropleth map | Barangay-level risk map colored by reported case density. Recommendation pins show where to deploy this week's intervention. |
| Cases averted chart | Projected cumulative cases averted versus a fixed-schedule baseline, with confidence range. |
| Plain-language rationale | 1–2 sentence explanation per recommendation in plain Filipino-English. Explains the key factor driving the decision (e.g. rainfall forecast, case trend, budget remaining). |
| Confidence / uncertainty flags | Low-confidence weeks flagged visually. Tells the officer when to trust the recommendation less. |
| Weekly update flow | Officer can enter new case data any week. Frozen policy re-runs with updated inputs. Plan refreshes. |

### v2 (thesis-complete)

| Feature | Description |
|---|---|
| PDF export | Printable A4 intervention plan for filing, sharing, and supervisor approval. |
| Shareable URL | Unique URL per plan version. Officer can share with supervisor without printing. |
| Partial observability (world model) | Agent uses DreamerV3-based world model to infer hidden epidemic state from noisy surveillance data. Replaces full-observability PoC agent. |
| Multi-seed validation display | Confidence ranges derived from agent uncertainty rather than simulator variance. |

---

## 06 User flows

### Flow 1 — Happy path (first use, season start)

| Step | Screen | System action | What the officer sees |
|---|---|---|---|
| 1 | Setup | Load page | Municipality pre-selected as Dasmariñas. Empty budget and week fields. |
| 2 | Setup | — | Officer enters season week (e.g. week 4), total remaining budget (e.g. ₱48,000). |
| 3 | Setup | Fetch rainfall from Open-Meteo | Rainfall forecast for the week displayed. Officer confirms or overrides. |
| 4 | Setup | — | Officer enters barangay case counts from their logbook. |
| 5 | Setup | Submit → FastAPI /recommend | Loading state. "Generating your plan…" |
| 6 | Plan | Render plan | Intervention calendar, choropleth map with pins, cases averted chart, plain-language rationale. |
| 7 | Plan | — | Officer reviews recommendation, reads rationale, notes which barangay to deploy to this week. |

### Flow 2 — Error case (budget too low for any intervention)

| Step | Screen | System action | What the officer sees |
|---|---|---|---|
| 1–4 | Setup | Same as Flow 1 | — |
| 5 | Setup | Submit → FastAPI /recommend | Agent returns: no intervention feasible within remaining budget. |
| 6 | Plan | Render warning state | "Your remaining budget is below the minimum cost of any intervention. The plan recommends holding budget for week N when case risk is projected to peak." Rationale explains why waiting is better than a partial deployment. |

### Flow 3 — Returning user (weekly update)

| Step | Screen | System action | What the officer sees |
|---|---|---|---|
| 1 | Plan | Load existing plan | Previous week's plan displayed. Current week highlighted in calendar. "New data available?" prompt. |
| 2 | Update | — | Officer enters new case counts for the week. Rainfall auto-refreshed from Open-Meteo. |
| 3 | Update | Submit → FastAPI /recommend | Loading state. "Refreshing your plan…" |
| 4 | Plan | Render refreshed plan | Updated intervention calendar, map, chart. Changes from prior week highlighted. |

---

## 07 Screen inventory

### Screen 1 — Setup

**When:** First visit, or when starting a new season.
**Purpose:** Collect the inputs the agent needs to generate the first plan.

| Element | Description |
|---|---|
| Municipality display | Pre-set to Dasmariñas, Cavite. Not editable in PoC. |
| Season week input | Number input, 1–26. |
| Budget input | Peso amount. Remaining for the season. |
| Rainfall display | Auto-fetched 7-day precipitation forecast. Editable override field. |
| Barangay case table | One row per barangay. Last week's values pre-populated. Officer edits changed values. |
| Generate plan button | Submits to FastAPI. Triggers loading state. |

### Screen 2 — Plan (weekly home)

**When:** Every Monday. The primary screen the officer lives on.
**Purpose:** Show the current week's recommendation and the season plan.

| Element | Description |
|---|---|
| Season progress bar | Week N of 26. Budget remaining. |
| Choropleth map | Barangay polygons colored by case density (lightest = lowest risk, darkest = highest). Recommendation pins for this week's intervention. Tooltip on hover: barangay name, case count, recommended action. |
| Intervention calendar | Scrollable week-by-week table. Current week highlighted. Columns: week, intervention type, target barangay, estimated cost. |
| Cases averted chart | Cumulative cases averted vs fixed-schedule baseline. Confidence band. Current week marked. |
| Plain-language rationale | Card below the calendar. 1–2 sentences explaining this week's recommendation. Key driver bolded. |
| Uncertainty flags | Weeks with low confidence flagged with a warning indicator in the calendar. |
| Update data button | Opens the Update screen. |

**Important:** The choropleth and recommendation pins are a **data visualization + spatial heuristic layer**, not RL agent output. The agent decides WHAT intervention and WHEN. The spatial heuristic ranks barangays by reported case count and pins the recommendation to the highest-risk barangay. This distinction must be visually communicated — the map legend or a footnote must make clear that barangay targeting is based on reported cases, not the RL model.

### Screen 2b — Update

**When:** Optionally, any week when new case data arrives.
**Purpose:** Accept new case counts and re-run the frozen policy with updated inputs.

| Element | Description |
|---|---|
| Barangay case table | Same as Setup. Pre-populated with last entered values. Officer updates. |
| Rainfall display | Auto-refreshed. Override available. |
| Refresh plan button | Re-runs FastAPI /recommend with new inputs. Returns to Plan screen. |

---

## 08 Data I/O model

### Inputs

| Input | Source | Entry method | Notes |
|---|---|---|---|
| Municipality | Pre-set | None — Dasmariñas fixed for PoC | |
| Season week | Officer | Manual number input | 1–26 |
| Remaining budget | Officer | Manual peso input | Single scalar |
| Barangay case counts | Officer's logbook | Manual table entry, pre-populated | ⚠ See assumption below |
| Rainfall forecast | Open-Meteo API | Auto-fetched on page load | 7-day precipitation. Officer can override. |
| Barangay shapefiles | PSA / PhilGIS | Static GeoJSON bundled with frontend | ⚠ Availability not yet confirmed — see Open Decisions |

**⚠ Data assumption — barangay case counts:** This product assumes the health officer has access to weekly case counts at the barangay level from their own health center logbook. In the Philippine system, the officer is part of the data collection chain — cases are reported to the RHU before they reach PIDSR. This assumption has not been validated with a real health officer and may not reflect all municipalities. If barangay-level data is unavailable, the choropleth degrades to municipality-level display only and the spatial heuristic cannot function.

### Outputs

| Output | Format | Description |
|---|---|---|
| Intervention calendar | Week-by-week table | Intervention type, target barangay, estimated cost, budget remaining |
| Choropleth map | Rendered map with pins | Barangay risk visualization + this week's deployment target |
| Cases averted projection | Line chart | Cumulative cases averted vs fixed-schedule baseline, with confidence band |
| Plain-language rationale | Text card | 1–2 sentences per recommendation. Key driver highlighted. |
| Uncertainty flags | Visual indicators | Low-confidence weeks flagged in calendar |

### The spatial heuristic (explicit specification)

The RL agent outputs: intervention type + week + budget allocation. It has no knowledge of barangays.

The spatial heuristic translates this to a barangay recommendation as follows:

1. Rank barangays by reported case count for the current week (descending)
2. Pin the recommended intervention to the highest-ranked barangay
3. If budget allows multiple deployments, assign sequentially down the ranking

This rule is simple, transparent, and auditable. It is not a model output. It must be labelled as such in the UI.

---

## 09 System architecture

### Overview

```
Browser (Next.js)
    │
    ├── UI layer (React components, Leaflet map, Recharts)
    │
    └── API routes (Next.js)
            │
            ├── GET /api/rainfall → Open-Meteo API
            │
            └── POST /api/recommend → FastAPI sidecar
                        │
                        ├── Frozen PPO policy (Stable-Baselines3)
                        ├── SEIR-SEI ODE simulator (scipy)
                        └── Spatial heuristic layer
```

### Components

| Component | Technology | Responsibility |
|---|---|---|
| Frontend | Next.js (React) | UI, routing, API orchestration |
| Map | Leaflet + GeoJSON | Choropleth rendering, recommendation pins |
| Charts | Recharts | Cases averted chart, confidence bands |
| Rainfall integration | Open-Meteo (via Next.js API route) | Auto-fetch 7-day precipitation forecast |
| RL inference server | FastAPI (Python) | Receives state input, runs frozen PPO policy, returns recommendation |
| ODE simulator | scipy.integrate.solve_ivp | SEIR-SEI simulation for projection and policy rollout |
| Spatial heuristic | Python (in FastAPI) | Translates municipality-level RL output to barangay recommendations |
| Barangay data | Static GeoJSON (PSA/PhilGIS) | Bundled with frontend, served as static asset |

### Deployment

**PoC:** Local development only. Both Next.js and FastAPI run on localhost.

**Portability:** Both services are containerized with Docker. A `docker-compose.yml` defines the full stack — `docker-compose up` runs the complete system. Deployment to any cloud provider requires no architectural changes, only environment configuration.

---

## 10 Constraints & dependencies

| Constraint | Detail |
|---|---|
| Timeline | July PoC deadline is an engineering milestone, not a research deadline. Thesis continues through next year. |
| RL engine readiness | The FastAPI sidecar requires a trained PPO policy. The product cannot be fully integrated until at least a baseline policy exists. UI can be built against a mock API in parallel. |
| Geographic scope | Single municipality — Dasmariñas, Cavite — for PoC. All barangay data, shapefiles, and calibration parameters are Dasmariñas-specific. |
| Data access | No PIDSR access for PoC. Case data entered manually by officer. DOH epi bulletins used for calibration only. |
| Rainfall API | Open-Meteo is free, no API key required, covers Philippines coordinates. No rate limits for PoC usage. |
| Agent observability | PoC agent is full-observability PPO. Partial observability is the thesis's claimed research axis; the inference method is held open (world model is the leading candidate — DECISIONS #15/#16). The hardened agent is a product-safe drop-in behind `/recommend` — no architecture change. |
| Spatial RL | The RL agent will not be expanded to barangay-level state in the PoC. Spatial recommendations are heuristic-only. Barangay-level RL is future work. |
| Budget granularity | Single scalar budget. Multi-category line-item budget is a v2 upgrade. |

---

## 11 Open decisions

### Product-only

| Decision | Status | Detail |
|---|---|---|
| Weekly barangay data availability | ⚠ Assumption unvalidated | Document assumes officer has weekly barangay case counts from their logbook. Needs validation with a real health officer before v2. |
| Barangay shapefile for Dasmariñas | ⚠ Source unconfirmed | PSA and PhilGIS listed as sources. Specific availability and GeoJSON format for Dasmariñas barangays not yet confirmed. Must be resolved before choropleth can be built. |
| Spatial heuristic definition | ⚠ Needs spec review | Current spec: rank by reported case count, assign top-ranked barangay. Edge cases (ties, zero-case weeks, multi-intervention weeks) not yet handled. |

### Research-product intersections

| Decision | Status | Detail |
|---|---|---|
| Confidence range method | ⚠ Method undefined — honesty risk | PoC commits to confidence bands on the cases-averted chart. With a full-obs PPO agent these can only be simulator-derived (variance across stochastic eval runs). **Caution:** that variance captures only Monte-Carlo noise, not parameter or structural uncertainty — which dominate and could be far larger — so a tight band risks manufacturing false confidence, directly against the trust north star. For the PoC, present the number as a *relative/directional* comparison vs. baseline, or widen the band with an ensemble over parameter draws. Never show a precise band on a number that could be off by an order of magnitude. |
| Plain-language explanation source | ⚠ Method undefined | PoC commits to plain-language rationale. Full explainability module (SHAP, attention, counterfactual) is open in AEGIS.md. For July, a template-based fallback may be needed: "Fogging recommended this week because reported cases in [barangay] are trending upward and rainfall forecast is high." Needs decision before UI copy is written. |
| WHERE + RL scope future | ⚠ Deferred (not the chosen axis) | The thesis chose **observability**, not spatial, as its contribution axis (DECISIONS #16), so the spatial heuristic layer is **not** being replaced and the architecture stays stable. This becomes live again only if a later research decision expands the agent to barangay-level state. |

---

*End of PRODUCT.md · This document is the product and engineering source of truth for Aegis. When a product decision changes, update it here and log it in DECISIONS.md. When an open decision is resolved, move it from §11 to the relevant section above.*