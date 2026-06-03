## 6. Review Protocol

Review is mandatory at every phase gate. Skipping review is a process failure.

### 6.1 Classification

- **(A) Must resolve** — a genuine correctness error: a clearly wrong result,
  a broken computation, a circular or tautological method, a result that
  contradicts a well-measured reference (§6.8). Only Category A blocks
  advancement.
- **(B) / (C) Advisory** — anything that is not a correctness error
  (completeness gaps, prose, style, clarity, "could be stronger"). Optional
  notes the reviewer may record; they do NOT block PASS. The executor may
  apply them at its discretion.

### 6.2 Review Gate

Every phase gate has exactly **one reviewer** — the critical reviewer
(`agents/critical_reviewer.md`) — except Phase 2, which is self-review by the
executor. There are no review panels, no separate adjudicator: the single
reviewer reads the artifact, checks correctness, and issues the PASS / ITERATE
call directly.

| Phase | Type |
|-------|------|
| 1: Strategy | 1 reviewer |
| 2: Exploration | Self-review |
| 3: Processing | 1 reviewer |
| 4a: Expected | 1 reviewer |
| 4b: 10% validation | 1 reviewer → human gate |
| 4c: Full data | 1 reviewer |
| 5: Documentation | 1 reviewer |

The reviewer has full context (artifact + conventions + RAG corpus) and may
query the experiment corpus to verify claims. Category A → fixer → re-submit.

### 6.3 Reviewer Focus

The reviewer's job is narrow: **are the results correct, are they
reproducible, and is anything clearly broken?** Block only on genuine
correctness errors (Category A); record everything else as optional advisory
notes that do not block PASS. The genuine correctness gates that remain in
force are the closure alarm bands, the fit-triviality / circularity gate, the
tautological-comparison gate, and the validation-target rule (§6.8).

#### 6.3.1 Evidence-Based Review

A verification claim should cite specific evidence. Acceptable: "Closure
chi2/ndf = 1.3/36 (p = 0.24) from results/closure.json — PASS". A bare
"Verified"/"Looks reasonable" with no number or reference is not enough to
clear a correctness concern — if the reviewer cannot cite evidence for a
"verified" claim, it was not actually verified.

#### 6.3.2 Reviewer Investigation Subagents

The reviewer may spawn a focused investigation subagent when a correctness
concern needs deeper analysis than reading the artifact allows (tracing a
value through scripts, checking systematic propagation in code,
cross-referencing a published method). **Contract:** provide a specific
question + file paths + what to look for; the subagent reads and reports only
(never fixes); integrate findings as cited evidence.

### 6.4 Review Focus by Phase

The reviewer checks correctness, reproducibility, and whether anything is
clearly broken. The genuine correctness gates below remain in force at the
phases noted; everything else is advisory.

- **Phase 1 (Strategy):** No correctness gates yet (targets defined, not
  tested). Reviewer sanity-checks that the plan is coherent.
- **Phase 2 (Exploration):** Self-review by the executor.
- **Phase 3 (Processing):** **Closure alarm bands** (Category A):
  chi2/ndf < 0.1 (suspiciously good — investigate inflation/tautology);
  chi2/ndf > 3 or any pull > 5-sigma (method failure); closure `passes: false`
  in machine output while text claims acceptable (misrepresentation).
- **Phase 4a (Expected Results):** Closure alarm bands (as Phase 3); fitted
  parameters that are circular or tautological with chain inputs; validation
  target check (§6.8). A variation with exactly-zero impact in every bin is a
  likely broken evaluation — verify the varied input is non-trivial.
- **Phase 4b (10% Validation):** Numbers in the AN match the latest
  machine-readable outputs (stale numbers are a correctness error); results
  compatible with Phase 4a expected; validation target check (§6.8).
- **Phase 4c (Full Data):** GoF of the primary configuration (chi2/ndf < 3,
  p > 0.01 — selecting smallest-error while ignoring poor GoF is a correctness
  problem); fit-triviality gate (chi2 ≈ 0 / circular inputs, see §3); fit
  pathologies (degeneracies, boundary hits) investigated, not silently worked
  around; validation target check, every result vs PDG/reference (§6.8);
  AN numbers match JSON.

#### 6.4.1 Completeness Review (Phases 1 and 4a)

At Phase 4a the executor produces a **systematic completeness table**
(`| Source | Conventions | Ref 1 | Ref 2 | This analysis | Status |`). The
reviewer may use it to flag gaps as advisory notes; a missing-but-justified
source is not a correctness error.

#### 6.4.2 Figure Review

The reviewer (or the executor at its pre-commit self-check) confirms figures
are not physically broken. Real-error red flags (Category A): data/MC
normalization mismatch (MC > ~20% off across the bulk — likely a
normalization bug/wrong cross-section/missing background); postfit data/MC
worse than prefit; empty bins where events are expected (or vice versa);
ratio panel globally off 1.0; efficiency/purity outside [0,1]; unphysical
shapes (negative yields, rising where it should fall); distributions that
should differ appearing identical; **insane uncertainties** — error bars
larger than the signal or spanning the y-axis (common cause: plotting derived
quantities without explicit `yerr`). Mechanical/visual issues (illegible
text, legend overlap, code variable names, axis ranges, ratio-panel gap) are
advisory unless they indicate a real error.

#### 6.4.3 Documentation Review (Phase 5)

The reviewer reads the AN as a standalone referee would and confirms the
physics is correct and the numbers are reproducible. Narrative quality,
prose, figure-scrolling completeness, and rendering polish are advisory.

**Tautological comparisons (Category A).** If "comparison to theory/MC" is
mathematically identical to "comparison to expected" (expected derived from
the same MC used as the theory comparison), the AN must not present them as
independent. Either remove the redundant comparison or state the tautology
and what independent information it provides. **Concrete alarms:**
chi2/ndf < 0.1 between corrected data and MC truth (a data/MC closure check,
not validation, when the correction came from that MC); chi2 identically
zero (algebraic circularity — investigate, see §3 fit triviality gate);
per-subset fits returning identical central values (fit not using the data).
True validation requires (a) an independent dataset, (b) published results
from a different analysis, (c) a different method on the same data, or (d)
theory not used in the analysis chain.

### 6.5 Iteration and Escalation

The reviewer → fixer loop repeats until every Category A finding is cleared.
If it cannot be cleared after a few tries, escalate to the human rather than
looping indefinitely. On an ITERATE verdict the fixer addresses the Category A
items and the reviewer re-checks them, recording the outcome in a short
re-review note for the audit trail.

### 6.5.1 Reviewer Scoping

A single reviewer makes the PASS / ITERATE call. The reviewer should not
dismiss a genuine correctness finding as "out of scope" when the fix is cheap
(re-running a Phase 4 script with modified parameters, producing an additional
figure, propagating a systematic through an existing chain). When several such
findings require re-running an earlier phase, batch them into a single
regression/fix iteration (§6.7).

### 6.6 Human Gate

Between Phase 4b and 4c for both measurements and searches. The human
receives a publication-quality compiled PDF (not markdown) with 10%
validation results and the unblinding checklist.

| Response | Meaning | Orchestrator action |
|----------|---------|---------------------|
| **APPROVE** | Sound; proceed to full data | Continue to Phase 4c |
| **ITERATE** | Issues within 4b scope | Fix, re-review 4b, re-present |
| **REGRESS(N)** | Fundamental issue from Phase N | Non-destructive regression (below) |
| **PAUSE** | External input needed | Wait for human input |

**APPROVE** means the methodology is correct and the 10% validation clean.
It does NOT approve the final result — that comes after Phase 5.

**Human review checklist:** event-selection yields match the reference
within ~10% (large discrepancies = selection bug, not physics); key data/MC
comparisons acceptable (none with MC ≈ 2× data); closure/stress tests pass
(failures understood, remediation adequate); systematic table covers the
reference sources; 10%/expected result agrees with the reference within
uncertainties (else understood); all figures interpretable with accurate
captions (conference-talk quality); total uncertainties reasonable vs
published (if much larger, source identified); **"where I wasn't sure"
items** — the agent flags physics decisions it was unsure of, and the human
reviews these explicitly. The last item is critical: a flag like "I chose
κ = 0.5 on GoF, but κ = 0.3 gives 40% better precision with worse GoF — a
judgment I'm not confident about" is more valuable than a silent choice.

**ITERATE** is for fixes that don't change the analysis approach (missing
systematics, validation, prose, figures); cycle until approval.

**REGRESS(N)** is for fundamental issues tracing to an earlier phase; the
human specifies the phase, the issue, and what must change (e.g. REGRESS(1):
strategy missed a dominant background; REGRESS(3): published analyses use a
better discriminant → implement and compare; REGRESS(4a): systematic
treatment wrong → revise, re-run 4b).

**Non-destructive regression protocol:**
1. Orchestrator spawns Investigator to assess cascade scope.
2. Investigator produces `REGRESSION_TICKET.md` (what must change in Phase
   N; which downstream artifacts are invalidated vs reusable; rework scope).
3. Executor creates **new artifact versions** (e.g. `STRATEGY_v2.md`), does
   NOT overwrite originals.
4. Each downstream phase re-evaluates: if unaffected, reuse as-is; if
   invalidated, re-execute reusing code/infrastructure where valid.
5. Executor produces a new AN version with updated narrative (cohesion below).
6. Affected phases re-reviewed by the reviewer.
7. Re-present to human.

**Post-regression AN cohesion.** The AN must tell a cohesive physics story
reflecting the CURRENT analysis — body text reads as if the current approach
was the plan from the start; the reader need not know it was restructured.
The Change Log carries the full audit trail. The AN is a physics argument,
not a process diary.

**PAUSE** is for needs the system cannot provide (additional MC, theoretical
calculation, expert consultation, unavailable data). The human specifies
what is needed and provides it when ready.

### 6.7 Phase Regression

**Regression must be triggered, not avoided.** Do not paper over upstream
problems. Concrete triggers (must not be rationalized away):
- Data/MC disagreement on observable or MVA inputs
- Closure test failure (p < 0.05)
- Stress test failure without successful remediation (see §3)
- Operating point instability
- Unexplained dominant systematic (single source > 80% of total)
- MC used for periods without corresponding simulation
- Result > 3-sigma from a well-measured reference value (see §6.8)
- Result with relative deviation > 30% from a well-measured reference,
  regardless of pull (see §6.8 — triggers calibration-first investigation)
- GoF toy distribution inconsistent with observed chi2 (observed chi2 falls
  outside the 95% interval of the toy distribution)
- Wholesale bin exclusion: flat-prior gate or similar excluding > 50% of
  measurement bins (binning/method problem, not a systematic — see §3)
- Two distributions that should be independent appear visually identical
- Systematic double-counting: two+ sources produce numerically identical
  per-bin shifts (same effect under different names — merge and document)
- Correlated combination error: when combining observables/methods, shared
  systematic sources (scale variation, generator choice) must be treated as
  correlated; treating them as uncorrelated artificially reduces the total

**Procedure:** (1) document issue, identify origin phase; (2) classify as a
regression trigger in the review artifact; (3) orchestrator spawns
**Investigator** (reads trigger → origin artifact → traces forward →
produces `REGRESSION_TICKET.md`); (4) fix origin phase → re-review → re-run
affected downstream phases.

**Upstream improvement cascade (applies to all fixes, not just regression).**
When any component is improved (better truth labeling, refined calibration,
updated selection), the executor must trace all downstream consumers and
re-evaluate: identify every script/figure/table/AN section that used the old
output; re-run each with the improved input, or document why it is
unaffected; update the AN; if a downstream result changes materially
(>1σ shift or qualitative change in ranking/selection), propagate further.
Re-running a script is cheap; stale results that contradict the improved
methodology are expensive. When in doubt, re-run.

**Timing:** Regression may be triggered at any phase gate or by the human
gate. After human approval and entry into Phase 4c, new issues found in 4c
or Phase 5 are Phase 5 iteration unless the reviewer classifies them as
regression triggers (§6.7 list), in which case they still trigger regression.

**Not regression:** Presentation issues (labels, captions, formatting) →
Phase 5 iteration.

**Upstream feedback (non-blocking):** Any executor may produce
`UPSTREAM_FEEDBACK.md` for issues an earlier phase missed; routed to the
next review gate.

### 6.8 Validation Target Rule

When Phase 1 defines validation targets (PDG values, published
measurements), these create binding review obligations.

**Tier 1 — Mandatory investigation (Category A).** Any extracted parameter
meeting **either** trigger blocks advancement until resolved:
- **Pull threshold:** pull > 3-sigma from a well-measured reference, OR
- **Gross deviation threshold:** relative deviation > 30%
  (|result − reference| / reference > 0.3), regardless of pull.

No single number cleanly separates "method problem" from "new physics"; 30%
is guidance, not gospel — the point is to trigger investigation. Use physics
judgment, with 30% as the default trigger for mandatory formal investigation.

**Tier 1b — Method-improvement investigation (Category B, upgradable to A).**
When the result deviates by 1–3-sigma (or 10–30% relative) with a **known
directional bias from the choice of method** (NLO vs NNLO, mean value vs
differential fit, LO vs resummed), the reviewer must check:
1. **Is a better method feasible** with available data/tools? (Differential
   fit when corrected differential distributions exist; NLL when
   coefficients are published.)
2. **Did the reference analyses use a better method?** Compare the
   extraction method to the Phase 1 reference table. Using a simpler method
   that every published analysis avoided is a **method parity failure** —
   Category A if committed to in the strategy, Category B if not discussed.
3. **Would the better method plausibly resolve the deviation?** Estimate the
   shift from published comparisons.

If all three are yes, require the better method as the primary result (or at
minimum a documented cross-check) before advancing. "A known limitation of
our method" is not sufficient when the better method is feasible. A
1.7-sigma deviation with a known directional cause is qualitatively
different from a fluctuation — the former can be reduced.

**Tier 2 — Calibration-first investigation.** When the method passes MC
closure but produces a deviant result on data, that is the classic signature
of a calibration mismatch (MC-derived corrections not matching data):

1. **Diagnose what needs calibrating.** Back-substitute: given observed data
   rates and the known reference, solve for the implied MC-derived
   parameters (efficiencies, corrections, scale factors); compare implied vs
   nominal. Back-substitution is a **diagnostic**, not a calibration — using
   back-substituted values as the correction is circular and forbidden as
   primary (see step 3).
2. **Check physical plausibility.** Are implied shifts consistent with known
   data/MC differences (resolution, tracking, alignment)? A 5% efficiency
   shift matching a 5% resolution difference is plausible; a 50% shift with
   no mechanism is a red flag.
3. **Calibrate from independent observables.** For each miscalibrated
   parameter, find an independent data-driven measurement, stating its
   **independence level**:
   - **Independent** (gold standard): from a control sample / auxiliary
     measurement / sideband with no dependence on the primary observable
     (e.g. d0 resolution from the negative-side d0 distribution; tracking
     efficiency from tag-and-probe on a known resonance). Measured in data
     AND MC; the data/MC ratio is the scale factor.
   - **Weakly dependent** (acceptable with documentation): from an
     anti-signal region or a correlated-but-not-identical variable; the
     second-order circular dependence must be quantified.
   - **Circular** (forbidden as primary): any method assuming the primary
     result to derive the correction (back-substitution). Usable for
     diagnostics/cross-checks/systematic ranges, NOT the central correction.

   Apply the independent calibration and re-run; the calibrated result is
   the measurement, and the uncalibrated-vs-calibrated difference is the
   calibration systematic.
4. **Parameters that cannot be independently calibrated:** use the MC value
   as central (not back-substituted); inflate the systematic to cover the
   MC-to-back-substituted range; document explicitly. Back-substitution
   informs the systematic range — its legitimate role.
5. **Verify magnitude closure.** Independent calibrations + inflated
   systematics must account for the full deviation. "Correcting the
   efficiency explains 40% of the bias" is acceptable IF the remaining 60%
   is covered by inflated systematics — the total must be large enough that
   the uncalibrated result falls within ~2-sigma, else a bias source is
   missing.
6. **Document the calibration chain** in the AN: uncalibrated result,
   identified miscalibrations, calibration procedure (independence level per
   parameter), calibrated result, residual calibration systematic, and the
   no-calibration result.

If the deviation **cannot** be explained by calibration (unphysical implied
shifts, or calibrated result still deviates): check for bugs, sign errors,
unit mistakes, wrong inputs; if none, document as a genuine tension and
consider downscoping the parameter. Do not claim new physics without
exhausting mundane explanations.

**Rationale:** Accepting large discrepancies with well-known values erodes
credibility; the burden of proof is on the analysis. A measurement where MC
closure works but data deviates is almost always calibration — the standard
HEP response is to calibrate, not panic.

**This rule applies at the review gate for Phases 4a, 4b, 4c, and 5.** Not at
Phase 1 (targets defined, not tested) or Phases 2–3 (results not yet
extracted).

---

## Appendix B: Minimal Artifact Checklist

### Per-phase artifacts

| Phase | Artifact file | Must contain |
|-------|---------------|-------------|
| Strategy | `STRATEGY.md` | Signal/background enumeration, selection approach, blinding plan, systematics categories, literature citations, reference analysis systematic table |
| Exploration | `EXPLORATION.md` | Sample inventory, data quality assessment, variable ranking, preselection cutflow, data/MC comparisons |
| Processing | `SELECTION.md` | Selection definition, region definitions, background estimates, closure tests, per-cut distributions |
| 4a: Expected | `INFERENCE_EXPECTED.md` | Systematic table with per-source detail, fit model or correction procedure, expected results, validation tests, systematic completeness table vs references, covariance matrix |
| 4b: 10% validation | `INFERENCE_PARTIAL.md` + `ANALYSIS_NOTE_4b_v1.md` | 10% observed results, post-fit diagnostics, GoF, draft analysis note with full structure |
| 4c: Full data | `INFERENCE_OBSERVED.md` | Full observed results, post-fit diagnostics, anomaly assessment, comparison to expected |
| Documentation | `ANALYSIS_NOTE_5_v{final}.md` + `results/` | Complete analysis note with all sections, figures, LaTeX math, machine-readable results |

### Analysis note completeness checklist

The Phase 5 analysis note must satisfy ALL of these. Each is Category A if absent:

- [ ] LaTeX math delimiters used throughout (`$...$`, not plain text)
- [ ] One subsection per systematic source (not just a summary table)
- [ ] One subsection per cross-check (not just a mention)
- [ ] Per-cut event selection with individual cut distributions and efficiencies
- [ ] MVA diagnostics when a classifier is used (ROC, score distributions, feature importance, data/MC on classifier output)
- [ ] Data/MC comparison for every selection variable (grouped in appendix grids)
- [ ] Fit diagnostic plots (NP pulls, GoF, post-fit data/model comparisons)
- [ ] Full covariance matrix (statistical + systematic) in appendix
- [ ] Machine-readable `results/` directory with spectrum, covariance, parameters
- [ ] Comparison to published data with quantitative metric (not just "consistent")
- [ ] `pixi.toml` has an `all` task reproducing the full chain
- [ ] Experiment log is non-empty
- [ ] All intermediate phase artifacts exist on disk
- [ ] **No empty sections:** every heading has ≥1 paragraph of prose before any figure or table
- [ ] **Numerical self-consistency:** every per-section systematic table matches the summary table; every derived quantity matches its inputs; every prose number matches its table entry; cross-check against `results/*.json`. Common failure: a fix cycle updates the summary table but leaves stale values in per-section tables, discussion paragraphs, derived-quantity calculations, or appendix completeness tables.

### Experiment log minimum content

Entries for at least:
- [ ] Data format discovery (branches, trees, event counts)
- [ ] Key parameter choices and their reasoning
- [ ] Failed approaches and why they were abandoned
- [ ] Any bugs encountered and how they were resolved

---
