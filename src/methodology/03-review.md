## 6. Review Protocol

Review is mandatory at every phase gate. Skipping review is a process failure.

### 6.1 Classification

- **(A) Must resolve** — blocks advancement.
- **(B) Should address** — weakens the analysis. Must fix before PASS.
- **(C) Suggestions** — style, clarity.

### 6.2 Review Tiers

| Phase | Type | Notes |
|-------|------|-------|
| 1: Strategy | 2-bot | Sets direction; physics errors propagate |
| 2: Exploration | Self-review + plot validator | Mechanical; figures validated programmatically |
| 3: Processing | 1-bot | External eye on closure/modeling |
| 4a: Expected | 1-bot+bib | Gates 10% validation; AN v1 has citations |
| 4b: 10% validation | 1-bot+bib → human gate | Draft AN polished; bibtex validated |
| 4c: Full data | 1-bot | Methodology already human-approved |
| 5: Documentation | 2-bot (rendering + bibtex) | Final product |

**2-bot:** Physics reviewer (parallel), then arbiter. **1-bot:** Critical
reviewer + plot validator (parallel); Category A → fixer → re-submit. The
plot validator joins 2-bot and 1-bot reviews at figure phases (2–5);
skipped at Phase 1. The physics reviewer receives ONLY physics prompt +
artifact. The plot validator performs programmatic code/data checks (red
flags are auto-Category A). **+bib (4a, 4b):** add a BibTeX validator
(verify citations against DOI/arXiv/INSPIRE). **Phase 5 2-bot:** rendering
reviewer (compiles/inspects PDF) + BibTeX validator (parallel), then
arbiter; plot validator also joins.

**No self-review fallback.** All phases except Phase 2 require independent
reviewer subagents. See `agents/` for full definitions.

### 6.3 Reviewer Framing

Key question: **"What would a knowledgeable referee ask for that isn't
here?"** Check external completeness, not just internal consistency. Before
concluding, the reviewer must answer:

1. Are all sources in the applicable `conventions/` document implemented or
   justified as inapplicable?
2. Do 2-3 reference analyses exist, and does this analysis match their
   systematic coverage?
3. If a competing group published next month, what would they have that we
   don't?
4. **Are uncertainties honest in BOTH directions?** Inflation as well as
   underestimation: a systematic taken from MC when a smaller data value
   exists; total uncertainty large enough that every validation passes
   trivially (all pulls < 0.5σ); a dominant systematic improvable by
   re-evaluating on data; the measurement must have resolving power
   (distinguish SM from a ±20% deviation at 2σ) or state that it does not.
5. **Were limitations accepted without evidence of attempted improvement?**
   "Single generator" without evidence of attempting PYTHIA 8 is Category B;
   same for uncalibrated variables and dominant systematics. Aim for the
   best achievable result, not the minimum passing one.
6. **Does every result have context?** Every extracted parameter compared to
   ≥1 independent reference with a quantitative pull/chi2; the AN states
   resolving power. A result "consistent with everything" is not informative.

"No"/"non-empty" without justification → Category A.

#### 6.3.1 Evidence-Based Review (mandatory)

Every verification claim must cite specific evidence. NOT acceptable:
"Verified"/"Confirmed"/"Checks out" without a number or reference; "Looks
reasonable" without stating what was checked; "Consistent with expectations"
without the expectation and metric. Acceptable: "Closure chi2/ndf = 1.3/36
(p = 0.24) from results/closure.json — PASS". Applies to ALL reviewer roles.
A review with no evidence is a failed review. Inability to cite evidence for
a "verified" claim signals it was not actually verified — go back and check.

#### 6.3.2 Reviewer Investigation Subagents

Reviewers may spawn focused investigation subagents when a concern needs
deeper analysis than reading the artifact allows (tracing a value through
scripts, checking systematic propagation in code, verifying
intermediate-output consistency, cross-referencing a published method).
**Spawn when** the concern requires reading >3 files or tracing a
computation through code; most review work should not need this.
**Contract:** provide a specific question + file paths + what to look for;
the subagent reads and reports only (never fixes); integrate findings as
cited evidence. Document why the investigation was needed and what it found.

### 6.4 Review Focus by Phase

| Phase | Primary Focus |
|-------|---------------|
| Strategy | Backgrounds, systematic plan, selection exploration, reference analyses |
| Exploration | Samples, data quality, data archaeology findings |
| Processing | Closure, approach comparison, cut motivation, MVA input modelling |
| 4a: Expected | Systematic completeness + implementation audit, closure alarm bands, formula audit, Phase 1 traceability, published overlay |
| 4b: 10% | Draft AN quality, consistency with expected, diagnostics, number consistency |
| 4c: Full data | GoF, viability, competitiveness, fit pathologies, number consistency |
| 5: Documentation | See §6.4.3 |

**Phase 1 (Strategy):** Backgrounds complete? Systematic plan covers
conventions? 2-3 reference analyses tabulated? Selection exploration plan
identifies ≥2 approaches for Phase 3 (or documents infeasibility)?

**Phase 2 (Exploration):** Samples complete? Data quality OK? Distributions
physical? **Data archaeology** (archived data): all weight/flag branches
checked for non-triviality? Pre-selection efficiency characterized? Strategy
revision inputs flagged?

**Phase 3 (Processing):** Background model closes? Every cut motivated by
plot? Cutflow monotonic? **Approach comparison:** ≥2 selection approaches
tried with quantitative comparison — a single approach without comparison is
Category A unless the Phase 1 infeasibility exemption applies. **MVA:**
data/MC on classifier OK? Alternative architecture tried? **MVA input
modelling:** all classifier inputs checked for data/MC agreement before
training — any input with chi2/ndf > 5 entering the classifier without
justification/calibration is Category A. **Closure alarm bands:**
chi2/ndf < 0.1 Category A (suspicious); chi2/ndf > 3 or any pull > 5-sigma
Category A (failure).

**Phase 4a (Expected Results):**

- **Systematic completeness:** every source from conventions + references
  implemented or formally downscoped? Signal injection/closure passes?
  Operating point stable (Category A if not)? MC coverage matches data periods?
- **Phase 1 traceability:** every Phase 1 commitment implemented or
  downscoped (cross-check COMMITMENTS.md if present).
- **Closure alarm bands:** chi2/ndf < 0.1 Category A (suspicious);
  chi2/ndf > 3 or any pull > 5-sigma Category A (failure); `passes: false`
  in JSON while text claims acceptable is Category A (misrepresentation).
- **Systematic implementation audit:** for each systematic, verify the
  variation input actually changes, impact is non-zero with expected sign,
  evaluation level consistent (gen vs reco). Exactly-zero impact in every bin
  without documented verification is Category A.
- **Formula audit:** for every formula transforming measured quantities into
  physics results, verify dimensional consistency, limiting cases (100%
  purity → no correction), and cross-check against the published reference.
  The first hypothesis for a closure failure is a formula bug.
- **Generator comparison:** if committed and downscoped, the justification
  must include evidence of attempted installation/generation.
- **Published overlay:** published reference results overlaid with chi2.
- **Statistical methodology audit (Category A if violated):** all chi2 with
  full covariance (diagonal chi2 with all p = 1.000 is a red flag); closure
  on independent MC (pull = 0.000 at all operating points indicates
  self-consistency, not closure); every variation justified by a
  measurement/published value (not arbitrary ±50%); per-systematic impact
  figures present; extraction uses a differential fit when the differential
  distribution + covariance are available.
- **Validation target check (§6.8).**

**Phase 4b (10% Validation):** Draft AN publication-quality? Results
consistent with expected? Diagnostics clean? **Number consistency:** AN
text/table values match latest machine-readable outputs — any discrepancy
> 1% relative is Category A (stale numbers). Validation target check (§6.8).

**Phase 4c (Full Data):**

- Post-fit diagnostics healthy? Anomalies characterized?
- **GoF of primary configuration:** chi2/ndf < 3 (p > 0.01) — selecting the
  smallest-error configuration while ignoring poor GoF is Category A.
- **Systematics re-evaluated on full data** (not just transferred from MC)?
  A data-evaluated systematic differing > 2× from MC must be discussed.
- **Viability check on ALL reported measurements** (primary and secondary) —
  any failing the 50%/30% criteria explicitly noted.
- **Competitiveness check** (multi-observable): any measurement with total
  uncertainty > 5× published precision flagged as non-competitive — is
  improvement feasible?
- **Fit pathologies:** near-degeneracies, boundary hits, or flat likelihood
  directions investigated (not silently worked around)?
- **Validation target check:** every result vs PDG/reference — pull
  > 3-sigma is Category A unless quantitatively explained (§6.8).
- **Number consistency:** AN text/tables match JSON — discrepancy > 1%
  relative is Category A.
- **Operating-point consistency:** if the primary config changed since 4a,
  verify (a) the change is stated in results (not just Change Log), (b)
  systematics re-evaluated at the new operating point or transfer justified,
  (c) the "primary" label consistent throughout. Cross-OP pulls are approximate.
- **Known-underestimate check:** for each systematic, compare to any
  independent cross-check (e.g. generator comparison); if the cross-check is
  > 2× the assigned systematic, discuss and adjust the total (see §3).
- **Conditional escalation to 2-bot.** Phase 4c normally gets 1-bot, but the
  orchestrator MUST escalate to 2-bot if ANY of: a result deviates from
  Phase 4a expected by > 2-sigma; a new regression trigger fires (§6.7); a
  systematic on full data differs from MC by > 3×; GoF pathological
  (chi2/ndf > 5 or < 0.05). The 1-bot is sufficient only when full data is
  consistent with expectations.

#### 6.4.1 Completeness Review (Phases 1 and 4a)

**Phase 1:** systematic plan covers conventions sources; reference analysis
table present. Omissions without justification → Category A.

**Phase 4a:** executor produces a **systematic completeness table**
(`| Source | Conventions | Ref 1 | Ref 2 | This analysis | Status |`).
Reviewer verifies row-by-row; MISSING without justification → Category A.
Cross-check conventions "required validation checks" against the artifact.

#### 6.4.2 Figure Review

**Code linter (batch, automated).** A pixi task greps plotting scripts for
mechanical violations of `04-output.md`: absolute `fontsize=N`, wrong
colorbar patterns, missing `hspace=0`, `figsize` violations, `ax.set_title()`,
`data=False` + `llabel`/`text` stacking. Runs before the agent; any violation
auto-Category A. It reads code, not figures.

**Visual validator (agent, reads rendered PNGs).** Reads every figure image
and assesses visual quality as a referee would. Each is Category A: text
illegible at rendered size (0.45\linewidth); legend overlap; colliding text
(e.g. "ALEPHSimulation Expected"); code variable names instead of
publication names; subplot layout unsuited to content; inappropriate axis
ranges; ratio-panel gap.

**Physics content check (Category A if failed)** — the visual validator AND
all prose reviewers check whether the physics makes sense, not just
formatting. Red flags: data/MC normalization mismatch (MC > ~20% off across
the bulk — likely a normalization bug/wrong cross-section/missing
background); postfit data/MC worse than prefit; empty bins where events
expected (or vice versa); ratio panel globally off 1.0; efficiency/purity
outside [0,1]; unphysical shapes (negative yields, rising where it should
fall); distributions that should differ appearing identical; **insane
uncertainties** — error bars larger than the signal or spanning the y-axis
(common cause: plotting derived quantities without explicit `yerr`, so
mplhep auto-computes sqrt(bin content), meaningless for non-count values).
Any of these presented without comment is Category A.

A blanket "figures look fine" is not acceptable — list each figure number
with status. Runs during each phase's review, not just Phase 5.

#### 6.4.3 Documentation Review (Phase 5)

Reviewer reads the AN as a standalone journal referee would: every
systematic with method + impact? Every comparison quantitative? Reproducible
from the note alone? Conventions covered?

**Physics narrative quality (Category B if weak)** — the "nodding physicist"
test: clear motivation (a physics question, not "hasn't been done with this
data"); logical flow (selection→corrections→systematics); well-chosen,
interpreted cross-checks; physical interpretation of the numbers (not just
"M_Z = 91.188 ± 0.004" but "consistent with the world average, precision
limited by the 4-point energy scan"); context in the broader landscape;
**equations** defining observable, correction, systematic evaluation, and
fit/extraction (zero equations is Category A); a resolving-power statement
after the final result.

**Rendering mechanical checks (Category A if failed), on the compiled PDF:**
zero unresolved cross-references ("??" — grep `ANALYSIS_NOTE.log` and scan
PDF; common cause is lost `\label{}` when merging figures); TOC page numbers
match actual pages (spot-check 3; off by >1 page means too few passes); no
raw LaTeX/markdown visible (`$\sqrt{s}$`, `\textbf{}`, `@fig:name`); title
renders math correctly; no `$\pm$` with visible dollar signs; all composite
figure panels legible at rendered size (zooming required → Category A).

**Consistency sweep (Category A if failed):** "primary" configuration label
consistent throughout (search PDF for "primary"); abstract numbers match
conclusion and results-table numbers to ≥2 sig figs; event counts identical
(not approximate) across cutflow, body, summary tables; chi-squared
covariance treatment stated and consistent (full covariance used when it
exists, else Category B).

**Figure-scrolling test (Category B if fails):** can the complete physics
story be understood from the figures alone? Every major step (data quality,
selection, corrections, validation, results, comparison) has ≥1 figure;
non-trivial methods without a visual explanation are gaps.

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

**2-bot:** Repeat until arbiter PASS. Warn at 3, strong warn at 5, hard cap
at 10; the arbiter should ESCALATE rather than loop indefinitely.

**1-bot:** Warn at 2, escalate to human after 3.

**Re-review documentation (mandatory).** On an ITERATE verdict, the fix
agent produces fixes and the phase is re-reviewed. The re-review MUST
produce a written artifact (`{PHASE}_REREVIEW.md` or `{PHASE}_REVIEW_v2.md`)
explicitly verifying each Category A/B finding from the original. "ITERATE →
fix → advance without re-review" is a process failure. Cycle until PASS;
every cycle produces a numbered artifact for the audit trail.

### 6.5.1 Arbiter Dismissal Rules

**The arbiter may NOT dismiss a reviewer finding as "out of scope" if the
fix requires less than ~1 hour of agent processing time.** Re-running a
Phase 4 script with modified parameters, producing additional figures, or
propagating a systematic through an existing chain are NOT out of scope.

When multiple findings independently require re-running an earlier phase,
this is **extra motivation** to address them together in a single
regression/fix iteration, not a reason to dismiss each.

**Every dismissal must be justified** with: (1) a concrete cost estimate
(agent-hours, not "significant effort"); (2) why the finding does not affect
the physics conclusion; (3) a commitment to a future phase if applicable.
"Requires reprocessing" / "out of scope" without a cost estimate is not
acceptable. Phase regression (§6.7) exists precisely for upstream work.

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
6. Affected phases re-reviewed with the same panel.
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
or Phase 5 are Phase 5 iteration unless the arbiter classifies them as
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

**This rule applies to all review tiers (1-bot, 2-bot) at Phases 4a, 4b, 4c,
and 5.** Not at Phase 1 (targets defined, not tested) or Phases 2–3 (results
not yet extracted).

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
