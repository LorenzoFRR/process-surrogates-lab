# Project Charter — Process Surrogates Study Lab

| Field   | Value                         |
|---------|-------------------------------|
| Version | 0.4 (draft)                   |
| Status  | Under discussion              |
| Owner   | Lorenzo Ferreira              |
| Updated | 2026-10-25                    |

> This document sets the overall direction of the project. It does not specify any one feature.
> Each SDD cycle (BRAINSTORM → DEFINE → DESIGN → BUILD → SHIP) should trace back to it.
> When a cycle teaches us something that changes the direction, update this charter and bump its version.

---

## 1. Vision

Learn, in depth, how **data-driven surrogate models** can complement conventional
process simulation for **reactors and whole flowsheets**, and, just as important,
**when they can be trusted**.

Learning is the main goal, but the project is **run like a real engineering project**:
it has a codebase that grows, a backlog, design decisions, tests and reviews. It also
raises questions and answers them with evidence, following process engineering,
data science, and software engineering best practices. The skills come from doing the
project properly, not from isolated exercises.

Central question:

> *Given simulated process data, how well can an ML surrogate reproduce the process,
> can we explain what it learned in physical terms, and does adding physics make it
> better or more trustworthy?*

## 2. Learning Objectives

The project has three pillars that build on each other: **fit**, then **explain**, then **constrain**.
Each objective is phrased as "I can…" so that it can be checked off at the end of a milestone.

### 2.1 Surrogate modeling (fit)
- [ ] I can choose and justify a surrogate family (linear/polynomial, GP, tree ensembles, NN) for a given data regime (size, dimensionality, smoothness).
- [ ] I can design validation that reflects real use: random vs. grouped vs. **extrapolation** splits, and leakage in time-series data.
- [ ] I can quantify predictive uncertainty (GP posterior, ensembles, conformal prediction) and check whether it is calibrated.
- [ ] I can compare surrogates on accuracy, cost, and data efficiency, not on a single metric.

### 2.2 Interpretability / XAI (explain)
- [ ] I can apply and critique SHAP, PDP/ICE, and permutation importance, and say where each one misleads (e.g. correlated features).
- [ ] I can run global sensitivity analysis (e.g. Sobol indices) and relate it to model explanations.
- [ ] I can **check explanations against process physics**: does the model's sensitivity have the expected sign and magnitude (temperature effect on rate, recycle effects, etc.)?

### 2.3 Physics-informed / hybrid modeling (constrain)
- [ ] I can build physics-based features (dimensionless groups, Arrhenius-type transforms, balance residuals).
- [ ] I can enforce or penalize mass and energy balance consistency in a surrogate and measure the effect.
- [ ] I can build a grey-box model (a known mechanistic structure plus a learned correction term).
- [ ] I understand PINNs well enough to judge when they suit a problem and when they are overkill.

### 2.4 Process engineering (always on)
- [ ] I can describe the anchor dataset's process in detail: units, recycles, key variables, dominant physics.
- [ ] I can do a physics sanity check of the data before modeling (balances, ranges, expected trends).

### 2.5 Software & ML engineering (always on)
- [ ] I can structure a data science codebase as a tested, installable package rather than a pile of notebooks.
- [ ] I can make experiments reproducible and comparable: config-driven runs, tracked results, pinned environments.
- [ ] I can work in a disciplined way: issues → branches → PRs → review, with CI and recorded decisions (ADRs).

## 3. Domain Scope

- **In:** reactors and kinetics (CSTR/PFR-type behaviour, temperature and conversion relationships), and
  **multi-unit flowsheets with recycles** (interactions between units).
- **Later, maybe:** separations, thermodynamic property surrogates.

## 4. Data Policy

- **One anchor dataset for the whole project.** It is public and simulated, covers a reactor
  inside a flowsheet, and is chosen in Milestone 0 (§5). Every later milestone builds on it.
- We do not run simulators ourselves.
- The anchor dataset gets a full provenance and process record in `docs/datasets/`: source, license,
  citation, generating model, process description, variable dictionary, and known caveats.
- Adding another dataset (e.g. to test generalization) is allowed only as a deliberate charter change.
- Raw data is not committed to git. It is downloaded by a script into `data/` (git-ignored).

## 5. How the Project Is Organised

The project uses three levels, each mapped to the tooling:

| Level | What it is | Tooling |
|-------|------------|---------|
| **Milestone** | A stage of the project that delivers a coherent capability and targets a few objectives from §2 | GitHub Milestone |
| **Feature** | A step to implement within a milestone (a pipeline, a model, an analysis, a report) | One agentspec SDD cycle; parent issue |
| **Task / spike / question** | Implementation work, time-boxed investigations, analytical questions to answer | gh issues (sub-issues of a feature where it makes sense) |

**Questions are first-class work.** When the project raises a question ("is this split leaking?",
"does the model respect the energy balance?"), it becomes an issue or spike, gets answered with
evidence, and the answer is recorded in the feature's docs or as an ADR when it shapes the design.

**Learning is captured as part of shipping.** Each feature's `/ship` lessons-learned section covers
both engineering lessons and *what I learned* about the methods and the process.

### Milestone 0 — Groundwork: dataset, method, knowledge setup

Milestone 0 contains three spikes, each ending in an ADR:

- **M0.1 Choose the anchor dataset** (criteria below)
- **M0.2 Choose the data science method** (§7.2)
- **M0.3 Decide where and how knowledge bases live** (§7.3)

Anchor dataset selection criteria. Survey public simulated datasets against these:

1. The process includes a **reactor** and at least one **recycle** / multi-unit interaction.
2. It exposes **physical variables** (flows, compositions, temperatures, pressures). Known kinetics or a published model is a bonus.
3. It has **enough samples and enough range** in the operating conditions (not just one steady state).
4. It supports a clear **input → output surrogate framing** (manipulated or disturbance variables → key outputs).
5. It has a **clear license**, documentation, and citation.

The M0.1 ADR records the dataset chosen and the trade-offs.

### Possible later milestones (not a commitment)
1. **Foundation:** data ingestion, validation, exploratory analysis, physics sanity checks, project scaffolding.
2. **Fit:** baseline surrogates with an honest validation design and uncertainty.
3. **Explain:** interpretability and sensitivity analysis, checked against physics.
4. **Constrain:** physics-based features, balance constraints, grey-box models, compared with the baselines.

## 6. Principles

1. **Depth over breadth.** Understand one method well before adding another.
2. **Physics first.** Sanity-check the data and expectations before making any ML claim.
3. **Honest validation.** Report where a model fails (extrapolation, regime changes), not only where it works.
4. **Engineering discipline.** Tested library code, notebooks only for exploration and reporting, CI, typed and linted code, PR-based workflow.
5. **Reproducible.** Pinned environment, scripted data download, config-driven experiments.
6. **Decisions are written down.** Significant, hard-to-reverse choices get an ADR.

## 7. Workflow & Tooling

### 7.1 Engineering workflow

- **Spec-driven development:** one agentspec SDD cycle per feature
  (`/brainstorm` → `/define` → `/design` → `/build` → `/ship`). Phase documents refer to this charter.
- **Tracking:** a GitHub repo, with Milestones for project stages and gh issues for features, tasks, spikes and questions.
  - Proposed labels: `feature`, `task`, `spike`, `question`, `adr`, `bug`, `docs`, `dataset`.
- **Stack (tentative):** Python, pandas/polars, scikit-learn, a GP library, PyTorch (for NN/PINN), SHAP, SALib, pytest. To be settled in the first `/design` and recorded as ADRs.

### 7.2 Data science method

SDD covers *how each feature is engineered*. A data science method covers
*the lifecycle of the analysis*. The two are layered:

- **Data science method (outer level):** its phases shape the milestones and define the quality checks.
- **SDD (inner level):** one cycle per feature within a phase.

**Working hypothesis: CRISP-ML(Q)**, which is CRISP-DM adapted for ML with explicit
quality-assurance steps in every phase. Its quality focus suits the central question of when
a surrogate can be trusted. Candidates to compare in spike M0.2: CRISP-DM, CRISP-ML(Q), TDSP, KDD/SEMMA/OSEMN.

Draft mapping, to be confirmed in the M0.2 ADR:

| Method phase | Project milestone |
|-------------------|-------------------|
| Business & data understanding | Milestone 0, Foundation |
| Data preparation | Foundation |
| Modeling | Fit, Constrain |
| Evaluation | Explain, plus quality checks in every milestone |
| Deployment / monitoring | Mostly out of scope (§8) |

Learning the methodology is itself a goal: the ADR should explain *why* the chosen method fits,
not just name it.

### 7.3 Knowledge bases

The agentspec KB system (`/create-kb <domain>`, with an optional `--validated` mode where every claim
is cited and fact-checked) can create new domains. The existing domains cover data engineering and
cloud only, with no process engineering or ML methods.

- **Policy: project KBs live in the repo**, are version-controlled, and are reviewed through PRs.
  By default agentspec writes new domains into the plugin's per-version install folder, which
  would be lost on plugin updates.
- **Mechanism (ADR-001, #4):** domains live in `.claude/kb/`, registered in
  `.claude/kb/_index.yaml`. `CLAUDE.md` declares it as a second KB root for reading and the only
  target for writing; agents that are delegated KB work get these rules in their prompt.
  Findings: `docs/research/kb-location.md`.
- **Build them when needed:** a domain is created when a milestone needs it, not up front.
- **Use `--validated` for foundational domains** that agents will treat as ground truth.
- KBs ground the agents. They are not study notes, but reviewing their content is part of the learning.

| Planned domain | When | Mode |
|----------------|------|------|
| Anchor dataset (process description, variables, caveats) | After M0.1 | validated |
| Data science method (the one chosen) | After M0.2 | light |
| Process fundamentals (reactors, flowsheets, balances) | Foundation | validated |
| Surrogate modeling | Fit | validated |
| ML interpretability | Explain | validated |
| Physics-informed ML | Constrain | validated |

## 8. Out of Scope (for now)

- Running our own process simulators (IDAES, DWSIM, Aspen…)
- Real plant or proprietary data
- Production deployment, APIs, dashboards
- Online / real-time control

## 9. Open Questions & Risks

| # | Question / risk | Notes | How it resolves |
|---|-----------------|-------|-----------------|
| 1 | **The anchor dataset may limit the physics pillar.** Since everything depends on one dataset, missing physical structure would block §2.3 later. | Mitigated by the Milestone 0 selection criteria. | Milestone 0 ADR |
| 2 | Steady-state vs. dynamic data? | A well-known candidate, the Tennessee Eastman Process (e.g. the extended Rieth et al. 2017 set), has a reactor inside a recycle flowsheet but is dynamic and fault-oriented, so the surrogate framing would be soft-sensor-like. | Milestone 0 |
| 3 | If no public dataset meets the criteria, do we relax §4 and allow our own simulations? | This would be a deliberate change of direction, and would bump the charter's major version. | Milestone 0 |
| 4 | Which learning resources go with each pillar (books, courses, papers)? | Optional, but useful for depth-first study. | Collect as milestones progress |
| 5 | Does the chosen data science method fit a one-person learning project, or does it add process overhead? | Adapt it rather than adopt it wholesale. | M0.2 ADR |
| 6 | ~~Can agentspec use KBs stored in the repo?~~ | **Resolved:** yes, via `.claude/kb/` and a `CLAUDE.md` rule (no plugin changes). | ADR-001 (#4) |

## 10. Change Log

| Version | Date       | Change |
|---------|------------|--------|
| 0.1     | 2026-10-24 | Initial draft |
| 0.2     | 2026-10-24 | One anchor dataset for the whole project; Milestone 0 with selection criteria; arcs replaced by milestones → features (one SDD cycle each) → tasks/spikes/questions; project framed as a real engineering project; added the software & ML engineering pillar (§2.5) and engineering principles |
| 0.3     | 2026-10-24 | Added data science method (§7.2, CRISP-ML(Q) as working hypothesis) and knowledge base policy (§7.3, KBs in the repo, built when needed); Milestone 0 expanded to three spikes (dataset, method, KBs) |
| 0.4     | 2026-10-25 | M0.3 resolved: project KBs live in `.claude/kb/` with a `CLAUDE.md` rule (§7.3); §9 item 6 closed |
