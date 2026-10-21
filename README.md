# Mechanism Generator

**An open toolkit for designing linkages and cam mechanisms to specification — built with the measurement, verification, and honest-evaluation discipline of an ML research project.**

**TL;DR:** a research engine + spec format (OMTS) + fail-closed verification tooling for designing mechanisms to industrial-style tolerances; 3 case studies run brief → measured failure → capability, every outcome recorded as data; the failure corpora became a lab for grading learned models against exact analytic methods — including a 17,930× out-of-distribution error collapse to 1.10× and a 10× search-cost win from learned initialization, all with pre-frozen scorecards and no goalpost-moving.

Most mechanism design is done by intuition, handbooks, and CAD iteration. This project asks a different question: *can mechanism design be made rigorous the way ML evaluation is* — frozen scorecards before runs, fail-closed instruments, recorded outcomes that are never rewritten, and measured, honest failure?

> **Where to see the full engineering record:** [Mechanismo-AI/mechanism-generator](https://github.com/Mechanismo-AI/mechanism-generator) — 100+ reviewed pull requests, every experiment staged as a frozen document, every outcome recorded as data.

---

## What it does

The toolkit takes a **task specification** (where a part must be carried, held, flipped; what tolerances apply) and searches for mechanisms that satisfy it:

- **OMTS** — an open, versioned task-specification format for mechanisms: poses, tolerances, timing/synchronization relations, collision geometry, and dwell demands, with unit-quantified quantities and fail-closed validation. Unsupported features are loudly rejected, never silently dropped.
- **A planar four-bar research engine** (with an experimental six-bar motion solver) that searches candidate mechanisms against three-pose tasks, with full candidate-level exports.
- **Verification tooling** that checks *composed lines of mechanisms* — master-cycle synchronization, per-task phase offsets, sampled collision occupancy, and timeline artifacts — all as pure-stdlib, fail-closed code.
- **Case-study-driven development**: real industrial-style tasks drive what gets built; each case study runs brief → recorded failure analysis → capability design → decision, with every scorecard frozen before any run.

## The engineering discipline (the part I'm proudest of)

Every experiment in this project follows a fixed protocol:

1. **Freeze before run** — the scorecard, predictions, and decision rules are committed to git *before* the experiment runs. Predictions are graded CONFIRMED/REFUTED as data; a refuted prediction is a finding, never a re-run.
2. **Fail-closed everywhere** — instruments abort loudly on tampered inputs, missing data, or ambiguous state. Several caught real bugs (and one content-corruption incident) before they reached main.
3. **Determinism as a first-class standard** — runs are recorded only if two independent executions are byte-identical (with an explicitly declared exception list); byte-verified pushes via git blob-SHA checks.
4. **Recorded outcomes are never rewritten** — honest failures are kept as data. A "no candidate qualifies" result is a completed experiment, not an error.
5. **Full review workflow** — branch → PR with a plain-language review card → CI matrix (Windows/Linux × Python 3.10/3.12, 12 checks) → human review and merge.

## The ML angle

Because the toolkit grades *candidate generators*, it became a natural laboratory for evaluating **learned models against exact analytic methods** — with real measurement discipline rather than benchmarks:

- **Measured the OOD generalization disease.** A learned per-groove Fourier-feature regression model (ridge, closed-form solve) matched analytic closed-form solutions in-distribution to ~8e-6 units, but degraded **17,930×** under design-distribution shift. The disease was measured three independent ways before any fix was attempted.
- **Diagnosed before treating.** A controlled re-probe classified the failure as *data-limited* (the design variable was absent from the function's inputs) — not a training hyperparameter problem.
- **Treated it with a design-conditioned representation** — adding the design channel as first-order inputs to the feature map collapsed the extrapolation error from 17,930× to **1.10×** (ratio of medians), with an interpolation class that never degraded.
- **Measured what the model is actually good at.** Even while the learned *answer* failed OOD, the learned *initialization* survived: warm-started searches won/tied 16/16 demands at roughly **1/10 the oracle calls** of random restarts — a clean separation of "coverage" claims from "latency" claims, with a written claim rule to keep them distinct.
- **Honest head-to-head bookkeeping.** The learned supplier is labeled by its measured capability (`SUPPLIER OUTPUT`), with a pre-written label-change event that has so far *not* fired — because the analytic closed form remains exact and instant on the tested family. Grading a model without moving the goalposts is the whole point. *(How this relates to the broader field — and the infrastructure gap it targets — is in the section below the case studies.)*

## Recorded case studies (outcomes as data)


| Case              | Task                                                                       | Recorded outcome                                                                                                                                                                                                                                                                                                       |
| ----------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Dwell station** | Hold a carried point within 0.3 mm for ≥50° of crank rotation at a station | Four-bar pool fails at 203× the demand (measured); a cam dwell-arc design meets it by construction, with a complete tolerance-zoning story (0.05 mm zoned class, guaranteed 53.8° width). Closed through to a spec-vocabulary contribution (dwell duration semantics).                                                 |
| **Gap transfer**  | Carry a part level across a 260 mm gap at packaging tolerance              | The six-bar solver family failed honestly at every iteration (recorded plateau ~6–27× tolerance); the family answer ended up a dual-cam + lever design that passes both pressure-angle gates inside a 600×600 mm envelope — with an exact profile-tolerance map showing where the design is and isn't manufacturable. |
| **Flipper**       | Flip a flat part 180° between stations                                     | Both recorded families fail on their own instruments (a groove-cam fails the pressure-angle class structurally; six-bars fail at the through-vertical transit). Recorded as honest analytic failure with full machine-readable evidence corpora.                                                                       |


These failures are load-bearing, not dead ends: each case study was required to emit a machine-readable evidence corpus *whether it succeeded or failed* — and those corpora (838 rows with exact analytic ground truth from the flipper alone) are the training and evaluation base for the ML results in the section above. The failure analysis is where the learned-model line was born.

## The unglamorous part (and why it's the point)

Data-driven mechanism design is an active, fast-moving field: the [LINKS dataset](https://github.com/ahnobari/LINKS) released a hundred million simulated planar linkages, MIT's [DeCoDE lab](https://decode.mit.edu/) builds learned synthesis models and design benchmarks, and UC Berkeley's [Co-Design lab](https://codesign.berkeley.edu/) published [MechaFormer](https://arxiv.org/abs/2508.09005), a transformer that generates mechanisms as sequences. That work advances the *models*.

This project deliberately builds what that literature doesn't provide: the boring, behind-the-scenes work of assessing a mechanism's actual, real-world feasibility. Boring as it may be, a generated mechanism is only as trustworthy as the specification and the instruments used to grade it. 

- **A versioned task-specification format** (OMTS), so "this mechanism must do X" means something precise and machine-checkable: poses, tolerances, timing and synchronization relations, collision geometry, dwell demands — with unit-quantified quantities and fail-closed validation.
- **Composition and verification tooling** — synchronized *lines* of mechanisms, master-cycle timing, per-task phase offsets, collision gating, timeline artifacts — the work of making several generated mechanisms cooperate in time and space, which is where industrial automation actually lives.
- **An honest grading framework** — pre-frozen scorecards, fail-closed instruments, recorded outcomes that are never rewritten, and head-to-head evaluation where the exact analytic method is allowed to win.

If data-driven mechanism design is going to move from demos to industrial practice, this is the plumbing it needs, and the field currently doesn't have it. That's the gap this project works — on purpose.

## Built with

Python (stdlib-only verification tooling), PyTorch + NumPy engine, YAML/JSON-schema spec validation, GitHub Actions CI. Apache-2.0.

## About this repo

This repo is a curated, recruiter-friendly view of the project. The full engineering record — spec drafts, job freeze documents, per-run evidence, recorded outcomes, and the complete PR history — lives in the [main repository](https://github.com/Mechanismo-AI/mechanism-generator) and the [community repo](https://github.com/Mechanismo-AI/mechanism-generator-community).