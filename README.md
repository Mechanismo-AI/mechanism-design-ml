# Mechanism Generator

**Designing linkages and cam mechanisms to a written specification, and grading the results the way an ML project grades a model: with the tests fixed before the run, and with failures kept as data.**

This is a research project, not a product. It has an open task-specification format, a planar four-bar search engine, and, since late October 2026, a private library that designs eight mechanism families and picks between them for a brief. It has not yet produced a mechanism that was built and tested in hardware. This page says what works, what does not, and how the work was done.

**[Try the four-bar playground →](https://mechanismo-ai.github.io/mechanism-design-ml/)** Drag a few target points and it designs a linkage that traces a path through them, in your browser.

**[Install the engine on your own computer →](https://github.com/Mechanismo-AI/mechanism-generator-classic)** A frozen, open-weight release of the research engine (v0.1.0a10), with step-by-step setup.


## What exists

- **OMTS, an open task-specification format for mechanisms.** A task says where a part must be carried, held or flipped, and to what tolerance, including timing, synchronization, collision geometry and dwell demands. Validation is strict: an unsupported feature is rejected loudly, never dropped silently.
- **A planar four-bar research engine**, plus an experimental six-bar motion solver. It searches candidate mechanisms against three-pose tasks, checks each one against physical and kinematic criteria, and exports every candidate with its evidence.
- **A mechanism selector (private).** Eight families: four-bar, geared five-bar, slider-crank, dwell linkage, plate cam, barrel cam, rocker cam and Geneva drive. Each has its own search and its own independent verifier, written with a different method from the solver. Given a brief, the selector collects every verified design, filters by the space the brief allows, and ranks by build-error robustness and then size. Designs from every family but the dwell linkage can be exported as DXF drawings.
- **Dynamics limits (private).** A brief can limit acceleration, jerk, motor torque, transmission angle, speed on arrival, path deviation and the carried part's clearance, over the whole cycle or only while the part is carried. The selector rejects designs over a limit.
- **Verification tooling** for lines of mechanisms that must work together: master-cycle synchronization, per-task phase offsets, and sampled collision occupancy.
- **An installable classic release** of the four-bar research engine, with its three proposal models as open weights, the OMTS tools and the full training code: [mechanism-generator-classic](https://github.com/Mechanismo-AI/mechanism-generator-classic).
- **A browser playground** (above) with a simplified four-bar path generator anyone can use without installing anything.
- **Recorded case studies** (below), each following the same loop: a plain-language brief, a measured failure of the current engine, a capability design, and a decision.

## What does not exist yet

- No mechanism from this project has been built or tested on a bench. Every result below is a kinematic or design analysis on a computer.
- Contact and pin loads while carrying, collisions between a mechanism's own links, and collisions between the part and the mechanism are not checked. Two families (the dwell linkage and the Geneva drive) have no dynamics analysis yet.
- The cam verifiers check station accuracy against the cam rebuilt from the design itself, so that check is not independent. Their pressure-angle, undercut and assembly checks are.
- The selector has no robustness floor: a brief whose only verified design is fragile still gets that design as its pick (one of 28 benchmark briefs, below).
- Everything is validated on constructed briefs, not on a collection of real customer briefs.
- The flip-station case is unsolved.

## Case studies

Each outcome below is a recorded result. None involves a built mechanism.

| Case | Task | What was recorded |
|---|---|---|
| Dwell station | Hold a carried point within 0.3 mm for at least 50° of crank rotation | The best four-bar candidates missed the stillness demand by 203.69×. A cam dwell-arc design analysis reaches a nominal 53.8° dwell width, with a guaranteed 52.7° at a 0.05 mm tolerance class. Later, the selector picked a barrel cam of 666 mm envelope radius that holds in all 40 simulated mis-built copies (±0.05 mm). A rocker cam a third that size also passes, but misses the hold in 5 of 40 copies. A design analysis, not a built part. |
| Gap transfer | Carry a part level across a 260 mm gap at packaging tolerance | Successive six-bar attempts stayed well above tolerance (as high as about 27× in the last recorded iteration). A dual-cam and lever design analysis then met its pressure-angle gate (worst case 29.7° against a 30° limit, a thin margin) inside a declared 600 × 600 mm envelope. The envelope is an assumption, not a derived requirement. Later, the selector picked a rocker cam of 217 mm envelope radius over a 286 mm slider-crank; the best four-bar it found needs 947 mm. |
| Flip station | Flip a flat part 180° between stations | Both families tried, a groove cam and six-bar linkages, failed on their own instruments. Recorded as unsolved, with the evidence kept as data. |

## Choosing the mechanism: what the benchmark shows

The selector was run on 28 constructed briefs: the three case-study briefs plus variations in span, tolerance, dwell, available space and carrier pose. Every pick passed its family's verifier and was graded on 40 copies built with ±0.05 mm errors. Results from the 29 October 2026 run:

- **Every brief gets a pick.** Barrel cam 11, rocker cam 9, geared five-bar 5, slider-crank 2, plate cam 1. Ranked by size alone, the rocker cam would take 20 of the 28.
- **Small and robust pull apart on dwell briefs.** Wherever it fits, the rocker cam is the smallest verified design, but on the briefs that ask for a hold it keeps only 22 to 87% of mis-built copies. The barrel cam that wins those briefs instead is 2.6 to 3.6 times larger.
- **An honest pressure angle costs size.** Holding cams to the textbook limits (30° for a sliding follower, 45° for a swinging one), where an earlier run allowed 60°, made every cam pick larger. The earlier cam results are withdrawn where they differ.
- **The space decides the family.** Pivots confined to a side wall pick the slider-crank, or the plate cam when a hold is needed; a tall narrow column picks the barrel cam; briefs whose carrier tilts go to the geared five-bar (90 mm on the tabletop brief against the four-bar's 244).
- **One pick should not be built as picked.** On a short, wide band with a hold, the only verified design is a rocker cam that keeps 30% of mis-built copies. The open question is whether the selector should refuse such a pick or let the brief set a floor.

A review of the work by fresh reviewers on 29 October found, among other things, a showcase cam running at a 61° pressure angle and a benchmark comparison skewed by an unequal search. Both were fixed before the run above, and the numbers on this page come from committed run records.

## The learned-model results, good and bad

The case studies produced machine-readable records of solved and failed tasks, which became a test bed for comparing learned models against exact analytic methods. All results below come from small synthetic problems.

**Learned model, extrapolation failure.** On one cam-groove family, a ridge-regression model with Fourier features matched the analytic solution in-distribution (median error 8.4e-6 units). Under a shift in the design variable, the median error rose to 0.15 units, about 17,930 times larger and 13 times the 0.015-unit tolerance. A controlled check traced this to missing information: the design variable was not an input to the model.

**The fix, on that family.** Adding the design variable as an input brought the extrapolation median down to about 9.2e-6 units, a ratio to the in-distribution error of about 1.10× (1.0983× recorded). This held on a small grid of 6 training cells and 4 extrapolation cells. It shows the cause was correctly identified. It does not show the approach generalizes to other mechanism families.

**Learned starting points.** Warm-starting a search from the learned model won or tied in 16 of 16 demands, using about a tenth of the oracle calls of random restarts (mean 22.0 against 211.5). That is a cost result. It does not show better answers.

**The honest part.** The learned model has never beaten the analytic closed-form solution on the tested family, so it is still labeled a supplier output, not a capability. In other parts of the project, learned components also did no better than random restarts. On a synthetic set of 60 four-bar position tasks, the published proposal models led to a usable design on 28; the best of 128 random proposals reached 53, and a portfolio with no learned models reached 60. That is one corpus and one seed, but it was enough to stop training the proposal models. My current view is that learning is most likely to help with choosing and combining mechanisms, not with the precise sizing that closed-form methods already do well.

## How experiments were run

Every experiment followed one protocol: the scorecard and decision rules were written down before the run, predictions were graded against the result, and a refuted prediction was recorded as a finding and never quietly re-run. Instruments fail loudly on bad inputs. Runs were recorded only when two independent executions matched byte for byte. This was useful for grading learned models honestly. It also produced a lot of process, and the next step is to point the same effort at building a mechanism that works.

## Next

Private work continues on a robustness floor for the selector, on loads while carrying, and on taking a practical brief through to a built mechanism. I’m happy to discuss it.

## Try it

The [playground](https://mechanismo-ai.github.io/mechanism-design-ml/) designs a crank-rocker four-bar whose coupler point passes through 3 to 6 points you place in a 200 × 140 mm work area. It animates up to three designs and reports the error at each point, the minimum transmission angle and every dimension, and it can copy a design out as JSON. It usually finds designs within a few hundredths of a millimetre in a second or two.

It uses a simplified solver written for the page (random valid designs, then Nelder–Mead refinement), not the project's research engine. It checks kinematics only: position, full crank rotation and transmission angle. The page is a single file, [`docs/index.html`](docs/index.html), and also runs offline if you save it. Slider-crank, cam and six-bar versions are planned.

![A four-bar linkage whose coupler curve passes through three target points, with the crank, coupler, rocker and ground links drawn and the error at each target labelled](images/four-bar-research-engine.png)

*A design from the research engine for a three-point task. It passed the engine's practical selection checks but not all of its preferred engineering thresholds, and it is not hardware-validated.*

## Run it on your own computer

[mechanism-generator-classic](https://github.com/Mechanismo-AI/mechanism-generator-classic) is the research engine as released in October 2026 (v0.1.0a10), republished as a frozen, installable package. It designs planar four-bar linkages for three target positions or poses, validates OMTS task files, and includes the full training pipeline. Its tests run on Windows and Linux with Python 3.10 and 3.12. With Python 3.12 and Git installed:

```sh
git clone https://github.com/Mechanismo-AI/mechanism-generator-classic
cd mechanism-generator-classic
python -m venv .venv
. .venv/bin/activate        # Windows PowerShell: .\.venv\Scripts\Activate.ps1
python -m pip install torch==2.13.0 --index-url https://download.pytorch.org/whl/cpu
python -m pip install ".[engine]"
mechanism-models download --directory models
mechanism-generate --models-directory models --targets -6 2 -2 6 0.5 3.5 --branches both --device cpu --headless --output_root runs/demo
```

The model download is about 22 MB and is checked against SHA-256 hashes. The release is not maintained; newer work is private.

## Related work

This project is a small part of a fast-moving field. The [LINKS dataset](https://arxiv.org/abs/2208.14567) provides a hundred million simulated planar linkages for data-driven design, and [MechaFormer](https://arxiv.org/abs/2508.09005) generates mechanisms as sequences with a transformer. Both advance the models. This project works on a different part of the problem: the written specification, and the checking that decides whether a generated mechanism can be trusted.

## About this repository

This is a public summary. The full engineering record is kept in a private repository, and I'm glad to share it on request. The project's mission and review standards are in the [community repository](https://github.com/Mechanismo-AI/mechanism-generator-community).

Python: the classic engine uses PyTorch and NumPy; the mechanism library uses NumPy and SciPy only; YAML and JSON-schema validation. This repository is Apache-2.0.

## How this was built

I built this with AI coding assistants (Claude, ChatGPT and Mistral). I set the direction, decided what to try next, and reviewed and merged every pull request. The assistants wrote most of the code and drafted most of the experiment documents. The main repository has more than 175 pull requests, and its commit history shows the assistants as co-authors. I'm saying so plainly because a reader should know which parts are my judgment and which are assisted execution.
