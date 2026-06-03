## 3. Analysis Phases

Five sequential phases. Within a phase, **parallelize where possible** —
sub-delegate independent tasks (MVA training, systematics, plotting). See §3a.5.

### 3.0 Artifact and Review Gates

**Every phase boundary is a hard gate.** Phase N+1 cannot begin until Phase N
has produced its artifact AND passed review (§6).

Gate protocol: (1) produce artifact → (2) update experiment log → (3) review
(§6) → (4) resolve Category A items → (5) advance. Artifact existence is a
precondition — never skip an artifact, even under context pressure; write it
and stop cleanly.

See §3a for orchestrator architecture. Phase CLAUDE.md templates (from
`templates/`) are the operational entry points agents read at runtime.

---

### Analysis Types

- **Search:** Signal/background, SR/CR structure, blinding (§4), limits or
  significance.
- **Measurement:** Corrected spectra, event shapes, extracted parameters.
  Staged validation replaces blinding (§4).

Where phase descriptions reference search concepts (SR, CR, S/B), measurements
substitute: fiducial region, sidebands, purity optimization.

---

### Analysis Philosophy

**Correctness above all else.** The most important property is the right answer
with honest uncertainties; no polish compensates for a wrong result. Spend
whatever time is needed. Prefer the more rigorous of two approaches; investigate
unexpected validation results until you understand WHY; verify a too-small/
too-large systematic independently; assume you are wrong when disagreeing with a
published value until you prove otherwise; compute both approximate and exact
versions when unsure an approximation holds.

**Never adjust parameters to match.** When a plot/number disagrees, investigate
WHY — do not tune until it matches. Fabrication patterns: tuning a cut until
data/MC agrees; dropping a systematic because "too large"; smoothing a band for
appearance; choosing binning that hides a discrepancy; selecting a data subset
that agrees better. Each LOOKS correct but IS NOT. The antidote is parameter
provenance: every parameter needs a justification determined BEFORE seeing its
effect. "This value makes the plot match" means it was tuned and the result is
fabricated — the #1 failure mode in AI-physicist collaborations, since training
rewards agreeable outputs. The goal is the most convincing physics result, not
passing gates.

**The "nodding physicist" test.** At every phase — including Phase 4a — ask:
"Would a physicist find every step well-motivated, every number well-supported,
every limitation honestly assessed with evidence improvement was attempted?" If
no, the artifact is not ready. A Phase 4a AN should read like a draft physics
paper, not a technical checkpoint.

**Solve problems, don't accept limitations.** When you hit a limitation, first
try to solve it:
- **Poor tagger purity** → alternative variables/thresholds, PID truth labels,
  contamination matrix correction. AUC 0.76 is workable with correction.
- **Single generator** → generate particle-level predictions from a modern
  generator (PYTHIA 8 Monash / HERWIG 7 at particle level for e+e-→hadrons at
  Z pole ≈ 30 min). Expected at Phase 4a; downscoping requires documented
  evidence that install/generation actually failed.
- **Uncalibrated variable** → attempt data-driven calibration (reweight, scale
  factors, published measurements) before a flat systematic.
- **Missing published data** → digitize figures, query the corpus, extract from
  HEPData, read the paper. Every measurement compared to something.
- **Dominant systematic** → seek a better evaluation method; decompose (norm vs
  shape); check if the variation is physical or a procedure artifact. A
  10×-larger-than-published systematic is almost always an evaluation problem.
- **Data/MC disagreement** → a calibration opportunity: find the source, derive
  a correction, verify it reduces disagreement.

Accept a limitation ONLY when (a) you tried ≥1 concrete approach, (b) it failed
for a documented, specific reason (not "it's hard"), and (c) solving it would
require multi-day computation or external resources you lack. A limitation
without evidence of attempted improvement is Category B.

**Every measurement needs context.** After any measurement: (1) compare to the
published reference/expectation, (2) state whether consistent and if not why,
(3) state what physics question it answers, (4) state what deviations it can
resolve. Insufficient: "The tracking efficiency systematic is 0.95%." Adequate:
the same number plus evaluation method, the conservative bound it derives from
with citation, consistency with the published range, and where the impact is
largest.

**Cross-checks are the physics content.** A measurement with three agreeing
cross-checks beats one with tighter errors and none. Proactively implement:
alternative correction method (IBU vs bin-by-bin), alternative selection (tight
vs loose), subperiod stability, published-result overlay with chi2, independent
theory comparison.

**Published result overlay is mandatory.** If published results exist for the
same/similar observable, extract numerical values and overlay them with a chi2.
Phase 1 extracts published values; Phase 4a must overlay them. Deferring to
Phase 5 is Category B.

**Implement improvements, don't defer them.** If a feasible improvement (better
tagging, generator comparison, calibration, published overlay) can be done in
< 2 hours, do it now. "Future Directions" is for genuinely infeasible items
(new data, new algorithms, multi-day compute) — NOT ~30-min PYTHIA runs,
~1-hour contamination corrections, or ~1-hour systematic decompositions. See §12.

---

### Phase 1: Strategy

**Goal:** Written analysis strategy a collaboration reviewer could approve.

**The agent must:**
- Query the experiment corpus for detector capabilities, prior work, datasets.
- Identify signal process, backgrounds (irreducible/reducible/instrumental),
  discriminating variables.
- Propose selection approach, background estimation strategy, control regions.
- **Technique justification:** defend the chosen technique against alternatives
  (double-tag vs single-tag, bin-by-bin vs full unfolding). The reader must
  understand why it was chosen over alternatives.
- **Selection approach exploration plan.** Phase 1 defines which approaches to
  explore, not which single one to use. Identify ≥2 candidate approaches and
  commit to trying both in Phase 3, with expected advantages, costs, and the
  comparison metric. Declaring a single approach is acceptable only if
  alternatives are documented infeasible (not merely suboptimal) AND the Phase 1
  review validates this.
- **Method parity with published analyses (binding).** Compare the proposed
  method — including the statistical extraction method itself — to reference
  publications. If they used a more sophisticated method (differential fit vs
  mean, NLO+NLL vs NLO, profile likelihood vs chi2), either (a) commit to the
  same/better method, or (b) justify a simpler method by a specific technical
  limitation AND commit to implementing the published method as a cross-check.
  "Easier" is not a justification. Reviewed at Phase 4a; silent downscoping is
  Category A.
- **Systematic plan:** read the applicable `conventions/` document, enumerate
  every required source with "Will implement" or "Not applicable because
  [reason]." Binding — Phase 4a reviews against it.
- Identify 2-3 reference analyses, tabulate their systematic programs, and
  **extract published numerical results** (central values + uncertainties at
  representative kinematic points) into the strategy artifact. These become
  binding comparison targets at Phase 4c (§6.8). Do not defer literature
  extraction to Phase 5.
- **Constraint and limitation labels.** Label constraints [A1]…, limitations
  [L1]…, key decisions [D1]…. These propagate to later phases and the AN. A
  constraint restricts what can be done; a limitation weakens the result; a
  decision is a deliberate choice with alternatives.

**For measurements additionally:** define observable(s), correction strategy,
prior measurements (validation target), theory predictions for comparison.
**Theory comparison independence:** the comparison generators must include ≥1
INDEPENDENT of the MC used to derive the correction. Comparing to the same MC
is a closure check, not a theory comparison. If none available, document as a
limitation and plan particle-level-only comparisons.

**Flagship figures.** Identify ~6 "money plots" (final spectrum with
uncertainties, response matrix, key data/MC, systematic breakdown, theory
overlay). Defined here, produced at highest quality in Phase 5; the list
propagates to the AN writing.

**Methodology diagrams.** Apply the figure-scrolling test: if a reader
scrolling the AN figures would hit a non-trivial method (correction chain,
sample merging, region definitions) with no visual explanation, plan a diagram.
List planned diagrams with target AN sections; produced in Phase 5.

**Artifact:** `STRATEGY.md`. **Review:** 2-bot (§6).

---

### Phase 2: Exploration

**Goal:** Characterize data, validate detector model, establish selection
foundation.

**The agent must:**
- Inventory samples (files, trees, branches, events, cross-sections).
- Validate data quality (pathologies, outliers, unphysical values).
- Apply standard object definitions (from corpus), verify data/MC agreement.
- Survey discriminating variables, rank by separation power.
- **Data/MC agreement on candidate variables.** For every variable that may
  enter a classifier or selection, produce a data/MC comparison and report
  chi^2/ndf. This feeds Phase 3's variable quality gate — flag poor agreement
  so Phase 3 can discard or calibrate before training any MVA.
- Establish baseline yields after preselection.
- **Published yield cross-check (pre-selected data).** When data was
  pre-selected at ntuple production ("aftercut", "skimmed"), the pre-selection
  efficiency is an unmeasured correction. (1) Obtain published luminosity and
  cross-section per energy point. (2) Compute N_exp = L_pub × sigma_pub. (3)
  Compare to observed N_obs. (4) f_presel = N_obs / N_exp. (5) Tabulate per
  energy point and year; flag energy dependence — if f_presel varies > 2% it
  biases lineshape/shape measurements unless corrected (a Phase 4 input). (6)
  If uncomputable (no published luminosity), document as a limitation and plan
  for Phase 4. This catches energy-dependent pre-selection and mismatched data
  periods early.

**Data discovery:** metadata first → small slice (~1000 events) → identify
jagged structure → document schema.

**Data archaeology protocol (archived/open data).** Discover unexpected
properties before Phase 3:
1. **Check all weight/flag branches for non-triviality.** Print unique values,
   range, mean for every possible weight/flag/quality branch. Non-trivial
   weights (not all 1.0) affect every downstream computation (e.g. a per-track
   weight 0.08-4.5 that biases the measurement if ignored).
2. **Check what processing was applied.** Compare counts to published σ×L; if
   lower, data was pre-selected — determine what was cut and the feasibility
   impact (e.g. leptonic events entirely absent from "aftercut" ntuples).
3. **Check MC generation parameters** (generator, tune, beam energy, process)
   match data-taking; document coverage gaps for Phase 4.
4. **Check for truth-level information** (gen quantities, particle-level
   definition, truth-matching). If absent, document alternatives.
5. **Strategy revision gate.** If any discovery materially changes feasibility
   (required branches absent, truth labels unavailable, dominant weight not
   understood, leptonic events pre-selected away), flag the exploration artifact
   as a **strategy revision input** stating what changed and the implications.
   The orchestrator re-reads and updates STRATEGY.md before Phase 3. This is the
   normal flow (not a regression); the revised STRATEGY.md gets a change-log
   entry and a lightweight 1-bot re-review of changed sections.

**PDF build test (independent):** stub `pixi run build-pdf` to verify the
toolchain. Can run in parallel.

**Artifact:** `EXPLORATION.md`. **Review:** Self-review (§6).

---

### Phase 3: Processing

**Goal:** Implement the strategy. Searches: selection, regions, background
estimation. Measurements: selection, correction chain. Phase 3 executes the
plan; it does not redesign it.

**Selection:**
- For multi-dimensional selection, exploit correlations between variables;
  document why if using a simpler approach. Read how reference analyses handled
  the same selection.
- **Approach comparison (mandatory).** Try ≥2 selection approaches and compare
  quantitatively with a common figure of merit on the same sample. Document
  what was tried, the FoM for each, and why the chosen one is preferred. Only
  exemption: Phase 1 documented alternatives infeasible AND the Phase 1 review
  validated it.
- **Input variable quality gate (MVA analyses).** Before training, produce a
  survey table for all candidate inputs:

  | Variable | Flavour discrimination | Data/MC chi^2/ndf | Decision |
  |----------|----------------------|----------------|----------|

  Each input must pass a discrimination check and a modelling check. Variables
  with data/MC chi^2/ndf > 5 are discarded unless (a) exceptional discrimination
  AND (b) a validated data-driven calibration is applied. A poorly-modelled
  input biases the data result even if MC closure passes (closure cannot catch
  data/MC mismodelling by construction). **When the majority of inputs fail**,
  the MVA may still be viable if (1) the BDT output data/MC disagreement is
  smaller than the input-level disagreement AND (2) data-driven scale factors
  with an appropriate systematic can be derived for the BDT output in a control
  region. This calibrate-then-use approach is standard b-tagging practice and
  must be attempted before rejecting a more-discriminating MVA. Rejection on the
  input gate alone — without evaluating output data/MC or attempting calibration
  — is Category B. The AN documents either the calibration attempt and result,
  or a quantitative argument that calibration is infeasible. The experiment log
  records intermediate training attempts; the AN documents the survey, the
  rationale, the final classifier, and the result.
- If MVA: train, validate, optimize. Train ≥1 alternative architecture. Try
  multiclass if >2 physics classes. Check data/MC on classifier output.
  Sub-delegate training (§3a.5). Diagnostic plots (ROC, scores, feature
  importance) are AN-bound — save for Phase 5.
- Every cut motivated by a plot; N-1 distributions preferred. Document
  sensitivity to cut variation. Cutflows monotonically non-increasing;
  unexplained increases warrant investigation.

**Regions (searches):** define CRs (background-enriched) and VRs (between CR
and SR, statistically independent).

**Background estimation (searches):** estimate SR yields from CRs. Closure
test in VRs (p > 0.05 or Category A).

**Correction infrastructure (measurements):**
- Data/MC comparisons for all variables entering the observable, resolved by
  object category (Category A if missing for unfolded measurements).
- Response matrix: dimensions, diagonal fraction, condition number, efficiency.
- **Early diagonal fraction check (gate).** Before the full chain, compute the
  response matrix on ~10K MC events and report the diagonal fraction. If < 50%,
  investigate — read how published analyses built their response matrix. A low
  diagonal fraction may indicate a methodological choice (wrong matching), not a
  fundamental limit. Do not conclude unfolding is impossible without checking.
- Closure + stress tests on MC. Failure (p < 0.05) is Category A.
- Prototype full chain on data.
- Binning justified (resolution, statistics, physics features).

**Validation failure remediation (measurements).** A failed required test
(stress, flat-prior, alternative-method closure) must NOT be accepted at face
value. Attempt **≥3 independent remediation approaches** before documenting as
a limitation; ≥1 attempt should use a **minimal-context subagent** (only the
problem statement and data, no prior failed attempts in context). Read how
published analyses solved it. Document all attempts in the experiment log.
Accepting a validation failure without remediation attempts is Category A.

**Wholesale bin exclusion is a red flag.** If a flat-prior gate or similar
excludes > 50% of measurement bins, it indicates a fundamental problem with
binning, regularization, or method — triggering mandatory re-evaluation of
binning/method. Exclusion is appropriate for edge bins with genuine kinematic
boundary effects, not central bins that are poorly constrained.

**Sensitivity optimization (when insufficient):** maintain `sensitivity_log.md`.
Explore qualitatively different strategies; document each and its limiting
factor. Stop at the goal or diminishing returns (3+ approaches, <10% relative
improvement).

**Method health assessment (measurements).** The artifact must include an
explicit "Is the method working?" section answering: (1) closure converges to
chi2/ndf ~ 1 with a stable plateau over ≥ 2 iterations/values? (2) stress test
passes at the level of expected data/MC differences (characterize resolving
power, not just at 50% tilt)? (3) flat-prior test leaves > 50% of bins with
< 20% shift? (4) alternative method viable (closure chi2/ndf < 5)? (5) are the
tests non-tautological? A closure applying the correction to MC reco and
comparing to MC truth is legitimate; one that is algebraically guaranteed
(BBB factors from sample X applied back to X, residual zero by construction)
has no diagnostic power. chi2/ndf < 0.1 or residuals exactly zero → check
whether the result follows from algebra, not data. If ANY fails, redesign the
method or adjust binning before Phase 4 (≥3 remediation attempts per failure).

**Closure test alarm bands (mandatory, non-negotiable; apply at Phase 3 AND 4a):**
- **chi2/ndf < 0.1:** Category A — suspiciously good. Investigate uncertainty
  inflation (loop accumulating variance), tautological construction, or same
  sample for derivation and testing. 0.01 is not "excellent closure."
- **chi2/ndf > 3 OR any single pull > 5-sigma:** Category A — method failure.
  Do not proceed. Acceptable responses only: (a) fix the bug, (b) redesign,
  (c) fix a flawed test, (d) formally abandon with documentation.
- **Closure `passes: false` in machine output while text claims acceptable:**
  Category A — misrepresentation. If closure fails, say so. The first
  hypothesis is always a code bug (sign/formula/variable), not physics.

Agents consistently rationalize catastrophic closures (0.01 "good", -287 sigma
"known limitation"). The spec does not permit this.

**Artifact:** `SELECTION.md`. **Review:** 1-bot (§6).

---

### Phase 4: Statistical Analysis

Three sub-phases. **Both measurements and searches follow 4a → 4b → 4c.**

#### Phase 4a: Expected Results

**Goal:** Systematics, statistical model, expected results on Asimov only.

- **Published input lookup protocol.** When the strategy commits to a published
  value [D] (e.g. luminosity from a cited paper), obtain the actual published
  values — do not derive from data. RAG failing to surface a table does not mean
  unavailable. Escalation: (1) query RAG with paper ID + "Table" + quantity,
  (2) `get_paper` for section structure then search, (3) fetch the PDF and read
  the pages, (4) escalate to the orchestrator as a **blocker**. Substituting a
  data-derived value for a committed published value is Category A.
- Evaluate experimental + theory systematics as rate/shape variations.
  **Variation sizing:** every variation motivated by a measurement,
  calibration, or published uncertainty — not an arbitrary round number.
  "±50% on background" is unacceptable unless 50% is the measured uncertainty;
  use the sideband-fit (e.g. ±12%) or MC-normalization (e.g. ±20%) value.
  Arbitrary inflations are Category A.
- **No borrowed flat systematics.** Borrowing values from another analysis as
  flat percentages is not propagation. Vary the source and re-run the affected
  chain for bin-dependent shifts. Flat estimates acceptable ONLY when (a) the
  source is confirmed subdominant AND (b) proper propagation is infeasible AND
  (c) the flat value is justified by a cited measurement — all three documented.
  A perfectly flat relative shift across all bins is physically suspect.
- **No inflated uncertainties** (as harmful as underestimation — makes pull
  checks pass trivially, destroys resolving power). (1) **Re-evaluate, don't
  transfer** — a 4c data-based evaluation takes precedence; an MC value > 2× the
  data value is inflation unless justified. (2) **No "conservative" rounding up**
  — 0.006 on data does not become 0.017 because MC gave 0.017. (3) **Resolving
  power check** — if the total can't distinguish the SM from a ±20% deviation at
  2-sigma, say so. (4) **Budget cross-check** — compare the total to naive
  sqrt(N) scaling from a published result; if much larger, find and verify the
  responsible systematic. (5) **Pull distribution sanity** — across ≥5
  independent quantities pulls vs references should be ~unit-Gaussian; all pulls
  < 0.5σ with large uncertainties signals inflation (~32% should exceed 1σ). At
  review the question is symmetric: too small AND too large.
- **Overcoverage investigation (mandatory when pull RMS < 0.7).** (1) Identify
  the dominant overcoverage source. (2) Estimate pull RMS without it; rising to
  ~1.0 confirms the culprit. (3) State explicitly the uncertainty is dominated
  by [source], possibly overestimated, limiting resolving power. Category B;
  Category A if not discussed in the AN.
- **Known-underestimate protocol.** When an independent cross-check shows the
  true variation is N× the assigned systematic: if N ≥ 2 the cross-check MUST
  replace or inflate the assigned value ("lower bound" labeling is insufficient);
  if at a different evaluation level, estimate the equivalent or assign the
  larger difference as a conservative upper bound and document; the AN states the
  source value, the independent variation, and whether the total incorporates it.
- **Extraction method hierarchy (parameter measurements).** Use ALL available
  information. Best→acceptable: (1) **differential fit** (default) — fit theory
  to the binned corrected distribution with full covariance; (2) **moment fit**
  — when the differential prediction is unavailable or a moment is cleaner;
  (3) **total rate / mean value** — only as a cross-check or when uncorrected.
  When a corrected differential distribution AND covariance exist, mean-value-
  only is a downscoping that must be justified; implement both with the
  differential fit as primary.
- Construct binned likelihood (Asimov data, systematic terms).
- Validate: NP pulls small, fit converges, results sensible.
- Signal injection tests (searches) or closure tests (measurements).
- GoF: chi2/ndf AND toy-based p-value (saturated model).
- **Fit boundary check.** After every fit, verify no fitted parameter sits
  within 1% of a boundary. A parameter at a boundary means the fit wants to go
  further — the uncertainty is a lower bound. Either widen and refit, or
  document saturation and flag the uncertainty. Applies to physics and nuisance
  parameters.
- For measurements: expected result from MC pseudo-data, never real data
  (see `conventions/extraction.md`); full covariance (stat + per-syst + total)
  + comparison to ≥1 theory prediction using full covariance.
- **MC-derived quantities require matching conditions.** Do not apply MC-derived
  efficiencies/corrections/scale-factors beyond the conditions they were derived
  from (periods, detector configs, kinematic regions) without an extrapolation
  uncertainty. If MC covers only a subset, restrict the measurement or justify
  applicability and size the uncertainty.
- **Per-systematic documentation depth.** Each source described in running prose
  covering physical origin, evaluation method (and its justification), numerical
  impact per result parameter, and interpretation (dominant/subdominant/
  conservative? caveats?). Document failed alternatives. Write as natural prose
  — NOT bold-labeled headings ("**Origin:**", "**Method:**").
- **Phase 1 traceability.** Before finalizing the table, re-read STRATEGY.md and
  verify every committed source/variation was implemented or formally downscoped
  (with a [D] label in the log). A committed systematic silently absent is
  Category A.
- **Zero-impact sanity check.** Any variation with exactly-zero (or < 0.01%
  relative) impact → verify the varied input is non-trivial (varied sample
  differs from nominal? branch/weight exists?). More often a broken evaluation
  than a negligible effect; document the verification or assign a cited value.
- **Systematic implementation self-check (mandatory, documented in artifact).**
  Per variation verify: (1) the varied quantity actually changes (print nominal
  vs varied — identical → bug); (2) impact non-zero in some bins; (3) impact has
  the expected sign/direction (wrong direction → likely sign error or wrong
  quantity); (4) evaluation level consistent (gen systematics at gen level, reco
  at reco — mixing produces uninterpretable shifts); (5) propagated not borrowed
  (mark `[borrowed]` and justify if flat). This exists because 4 of 8
  systematics in one analysis had implementation bugs.
- **Generator comparison as concrete default action.** For hadronic observables
  at the Z pole, generating standalone particle-level predictions from ≥1 modern
  generator (PYTHIA 8 Monash, HERWIG 7, Sherpa) is an expected Phase 4a
  deliverable (~30 min for 1M e+e-→Z→hadrons at particle level). If archived MC
  is the only generator for corrections, this is the only genuine hadronization
  model dependence estimate. Downscoping to a literature-based systematic is
  acceptable ONLY if installation genuinely fails (documented error messages).
- **COMMITMENTS.md (mandatory tracking artifact).** At Phase 1 completion, list
  every Phase 1 commitment (systematics, cross-checks, validation tests,
  comparison targets, planned figures) with machine-readable status:
  ```
  - [x] D1: EEC self-pair convention (i<j) — resolved Phase 1
  - [D] D11: PYTHIA 8/HERWIG 7 standalone — downscoped Phase 4a
        (PYTHIA 8 installation failed: [error])
  - [ ] A1: Published ALEPH EEC overlay — NOT YET ADDRESSED
  ```
  Update at every phase boundary. At Phase 5 review every line must be `[x]` or
  `[D]` (formally downscoped with justification). Any `[ ]` at Phase 5 is
  Category A.
- **Phase 4a as draft physics result.** A physicist picking up the 4a AN must
  understand what is measured and why (with equations defining the observable
  and extraction/correction), how it compares to published values (overlay +
  chi2), the dominant uncertainties and whether well-motivated (physical origin,
  not "tracking: 0.95%"), what limitations exist and what was attempted, and
  whether competitive (explicit resolving-power statement). Minimum content:
  equations for observable/correction/systematic evaluation; a published-overlay
  comparison; a systematic completeness table with physical motivations; a money
  plot with total uncertainty band.

**Artifact:** `INFERENCE_EXPECTED.md` + `ANALYSIS_NOTE_4a_v1.md` (complete AN
with all detail using expected-only results; 4b/4c update numbers, Phase 5
polishes prose and re-typesets).

**Number consistency gate (every AN compilation, 4a/4b/4c/5).** Before
compiling any AN version, verify all numerical values match the latest
machine-readable outputs (inference artifacts or `results/`). Systematics,
central values, event counts, efficiencies — all current. Any discrepancy > 1%
relative is Category A. Prevents stale numbers (e.g. a systematic revised
9.3%→1.7% but the abstract still quoting the old value).

**PDF compilation is mandatory at 4a.** The AN is compiled to a
publication-quality PDF before review. The executor performs the statistical
analysis, writes the AN prose, AND typesets it in a single role: markdown →
`.tex` (pandoc) → `postprocess_tex.py` (deterministic structural fixes:
margins, abstract, references, table spacing, FloatBarrier, needspace,
duplicate headers, appendix, clearpage) → executor does figure composition and
longtable conversion → compile to PDF. The 1-bot+bib panel reads the PDF, not
the markdown.

**Review:** 1-bot+bib (§6).

#### Phase 4b: 10% Data Validation

**Goal:** Reality-check with a 10% subsample + update the AN with 10% results.

- 10% data (fixed seed), MC normalized to 10% luminosity.
- Run the full chain; evaluate GoF, NP pulls, impact ranking.
- Compare to Phase 4a expected (overlay, chi2); document discrepancies.
- For extraction: include diagnostics sensitive to data/MC differences (not
  just the final quantity). See `conventions/extraction.md` check #5.
- **Update the AN** (established in 4a) with 10% data results.

**Artifact:** `INFERENCE_PARTIAL.md` + `ANALYSIS_NOTE_4b_v1.md`.

**Figure reference verification (mandatory before PDF compilation).** Verify all
figure references resolve:
```bash
grep -oP 'figures/[^)]+\.pdf' ANALYSIS_NOTE_4b_v*.md | sort -u | \
  while read f; do [ -f "$f" ] || echo "MISSING: $f"; done
```
Copy/symlink any missing figures from earlier phases. A missing figure is
Category A — do not compile until all references resolve.

**PDF compilation is mandatory at 4b.** The human gate requires a
publication-quality PDF (pandoc → `postprocess_tex.py` → figure composition +
longtable conversion → compile). The human reviews the PDF, not markdown.

**Review:** 1-bot+bib (§6) → **human gate** (§4.2).

#### Phase 4c: Full Data

**Goal:** Final results on the full dataset.

- Full chain, post-fit diagnostics.
- Compare to **both** 10% and expected. Flag > 2-sigma disagreement with
  expected or disagreement with 10% beyond statistical scaling.
- Investigate anomalies (large NP pulls, poor GoF).
- **Re-evaluate systematics on full data.** The Phase 4a MC budget is a
  starting point. For each source involving a scan/comparison across
  configurations (kappa values, binning, alternative methods), re-evaluate on
  full data. If the data evaluation differs from MC by > 2×, document and
  justify which is used. Transferring the entire budget from MC to data without
  validation is a borrowed flat systematic — same prohibition applies.
- **Configuration selection must include GoF.** When multiple configurations are
  evaluated, the primary must satisfy statistical precision AND goodness-of-fit.
  A configuration with chi2/ndf > 3 (p < 0.01) must not be primary without
  (a) investigating the poor GoF, (b) documented justification it does not
  invalidate the result, (c) explicit comparison to acceptable-GoF
  configurations. Selecting solely for small statistical error while ignoring
  fit quality is Category A. If best-precision has poor GoF, fix the model,
  choose an acceptable-GoF configuration, or quote that one with the poor-GoF
  as a cross-check.
- **Fit pathologies must be investigated.** Degeneracies, near-singular
  Hessians, near-flat likelihood directions, or boundary-hitting parameters are
  anomalies, not configuration choices. If a fit that worked on MC develops a
  pathology on data, document what changed, why, and whether it could affect the
  chosen configuration at higher statistics. Silently switching configurations
  without investigation is Category B.
- **Fit triviality gate (mandatory).** If the fit chi^2 is identically zero (or
  within numerical precision of zero), OR the fitted parameters exactly equal
  the input assumptions used elsewhere in the chain, STOP and investigate
  whether the methodology is algebraically circular. chi^2 = 0.000 is not
  "excellent fit quality" — it is an alarm. Before proceeding: (1) trace every
  input to the cross-section formula (where does each luminosity, efficiency,
  background fraction come from?); (2) if any input was derived using the same
  theoretical cross-section the fit measures, the fit is circular; (3) if
  circular, seek independent inputs (published luminosities, external efficiency
  measurements) — check the cited papers' tables; (4) only after exhausting
  alternatives may the analysis proceed with circular inputs, and then the
  results must be presented as "self-consistency check" values, not
  measurements. Applies even if the executor already acknowledges the
  circularity — acknowledgment is not remediation. A circular measurement when
  non-circular inputs exist is Category A.
- **Poor GoF → investigate calibration (mandatory).** When chi2/ndf >> 1 (e.g.
  > 10) appears after introducing a previously-absent correction (derived →
  published luminosities, adding efficiency, changing background model), the
  first hypothesis is a **missing calibration** (energy/time/sample-dependent
  effect previously absorbed by the derived quantity). Do NOT conclude "the
  correction doesn't work" or revert to the circular approach. Instead: (1)
  examine residuals per point — which drive the chi2; (2) check if problematic
  points share a property (energy, year, detector config); (3) compute observed/
  expected yield ratios per point — a smooth non-flat ratio indicates an
  unmeasured varying efficiency; (4) calibrate the missing efficiency using
  published reference values per point; (5) refit — chi2 should improve
  dramatically; if not, the problem is elsewhere.
- **Viability check (every reported measurement — primary, secondary, derived).**
  If a secondary fails, either (a) investigate and explain, (b) label
  non-competitive with the failed criterion in the results text, or (c) remove
  it. Silently omitting the check for a secondary is Category B. Verify: total
  uncertainty > 50% of central value or > 10× world-average precision may lack
  resolving power (document what it can distinguish); central value deviating
  > 30% relative from a well-measured reference triggers §6.8 (when MC closure
  passes but data deviates, first hypothesis is a calibration mismatch — follow
  the §6.8 calibration-first protocol); intermediate unphysical values
  (negative widths, imaginary couplings) → document as "not reliably
  extractable" rather than quote with inflated uncertainty. Quoting a result
  with > 3-sigma pull OR > 50% relative deviation from a well-measured value
  without a quantitative explanation is unacceptable (§6.8).
- **Competitiveness assessment (multi-observable analyses).** Each quantity
  assessed for competitiveness. If total uncertainty (stat ⊕ syst) exceeds the
  published reference precision by > 5×, it is non-competitive — requiring a
  decision: if the dominant systematic is methodological and an alternative is
  feasible (< 2 hours), attempt the improvement before 4c; if genuinely
  data/detector-limited, label "non-competitive" and state what would make it
  competitive. Non-competitive results may be reported but not presented as
  competitive — "consistent with the SM" is vacuous when error bars span the
  interesting range.

**Machine-readable results (mandatory).** The executor creates
`phase5_documentation/outputs/results/` with JSON files for all numerical
results: fitted parameters with uncertainties, derived quantities, per-source
systematic shifts, covariance matrices, per-energy-point cross-sections. These
are the single source of truth — the AN reads from these files, not prose.

**Artifact:** `INFERENCE_OBSERVED.md` + `ANALYSIS_NOTE_4c_v1.md`.

**AN update is mandatory at 4c.** Update the AN with full data results
(replacing 10% numbers). **PDF compilation is mandatory at 4c** — compile to
PDF before review (pandoc → `postprocess_tex.py` → executor typesetting →
tectonic). Every AN-producing phase (4a, 4b, 4c, 5) compiles to PDF.

**Review:** 1-bot (§6).

---

### Phase 5: Documentation

**Goal:** Final analysis note — publication-quality, self-contained, 50-100 pages.

Phase 5 requires figure production, AN prose writing, and PDF typesetting —
sequential, dependent concerns (figures → prose → PDF). The executor performs
all three in a single role.

**Figure production.** Produces remaining AN-specific figures not generated in
Phases 2-4 (per-cut distributions, per-systematic impact plots). Reads data,
runs plotting scripts, saves to `phase5_documentation/outputs/figures/`.
**Flagship figures** (from Phase 1) get extra attention (tighter axis limits,
careful legends, considered colors). **Methodology diagrams** from Phase 1
(correction chains, region schematics, flow charts) are produced here — see
`appendix-plotting.md`.

**AN writing.** Reads ALL phase artifacts (strategy, exploration, selection,
inference) and the figures directory; writes the complete AN to
`ANALYSIS_NOTE_5_v1.md`. Reads artifacts and writes prose — no data files or
code. Must meet the completeness test: a physicist unfamiliar with the analysis
can reproduce every number from the AN alone.

**Interpretive quality.** Every section passes the "nodding physicist" test:
- **Minimum 4 equations:** observable definition, correction procedure,
  systematic evaluation, fit/extraction model. Zero equations is Category A.
- **Every result has context:** 2-3 sentences after each results table/key
  figure interpreting the numbers physically, comparing to published values,
  stating consistency and why.
- **Validation summary table:** every test (closure, stress, stability,
  cross-checks) with chi2/ndf, p-value, verdict.
- **Resolving power statement:** after the final result, what deviations the
  measurement detects at 2-sigma.
- **Published overlay:** ≥1 figure overlaying measured values with published
  results and a chi2 annotation.

**Number consistency gate.** Before PDF compilation, verify all AN values match
the machine-readable JSON in `results/`. Any discrepancy > 1% relative is
Category A.

**Figure composition annotations (mandatory).** The executor knows which figures
are related (it writes the prose around them). When referencing a group
(per-variable data/MC, per-systematic impact maps, per-cut distributions,
nominal+uncertainty pairs), annotate the grouping in markdown with HTML comments
read back during typesetting:

```markdown
<!-- COMPOSE: 2x3 grid -->
![Charged multiplicity...](figures/datamc_nch.pdf){#fig:datamc-a}
![Visible energy...](figures/datamc_evis.pdf){#fig:datamc-b}
...

Pre-selection data/MC comparisons for the six key kinematic variables.
Agreement is within 5% across all variables...
```

Annotation syntax: `<!-- COMPOSE: NxM grid -->` merges the next N×M figures into
one multi-panel figure with a shared caption; `<!-- COMPOSE: side-by-side -->`
merges the next 2 as (a)/(b) panels; `<!-- COMPOSE: 1xN row -->` a single-row
layout; `<!-- FLAGSHIP -->` full-page standalone treatment. This grouping is a
physics judgment. Annotations persist across versions (4a→4b→4c→5) since the
executor updates rather than rewrites; the merge scan (below) is a safety net.

**Typesetting.** Run AFTER the AN prose is written:
- `pandoc` markdown → `.tex` (not PDF):
  `pandoc ANALYSIS_NOTE_5_v1.md -o ANALYSIS_NOTE_5_v1.tex --standalone
  --include-in-header=../../conventions/preamble.tex --number-sections --toc
  --filter pandoc-crossref --citeproc`
- `postprocess_tex.py` handles deterministic structural fixes (title math
  sqrt(s)→$\sqrt{s}$, escaped standalone math, margins, abstract→environment,
  references unnumbering, table spacing, short longtable→table conversion,
  FloatBarrier, needspace, duplicate header/label removal, appendix insertion,
  clearpage, stale phase label warnings).
- The executor then reads the `.tex` and does judgment work:
  - **Combine related figures** into `\begin{figure}` environments with
    side-by-side `\includegraphics` separated by `\hspace{0.01-0.02\linewidth}`.
    Do NOT use `\subfloat` — use unified captions with (a)/(b)/(c) labels.
    Primary source: the executor's own `<!-- COMPOSE -->` annotations (search
    the markdown for `COMPOSE`); `<!-- FLAGSHIP -->` gets standalone full-page
    treatment. **Mandatory merge candidates** (Category A if left standalone):
    per-variable distribution surveys (8 inputs → one 2x4/3x3 grid);
    per-systematic shift/impact maps; per-cut before/after comparisons;
    per-subperiod/per-category comparisons; sequential figures differing only in
    one parameter; **nominal + uncertainty pairs** (a 2D map followed by its
    uncertainty/relative-uncertainty map → (a)/(b) panels in one figure). The
    rule: if N consecutive figures share axes/layout and differ only in a
    label/parameter, merge them. Scan for runs of 3+ sequential related figures,
    and for any figure immediately followed by its uncertainty counterpart
    (captions/filenames with "uncertainty", "error", "sigma", "stat_unc",
    "rel_unc"). Write one composite caption per merge.
  - **Convert longtable to table** — no column overflow; use `\resizebox` or
    `\small` for wide tables.
  - **Verify every section has prose** — no bare headings before figures.
  - **Check caption quality** — flag captions under 2 sentences.
- **Compile → read → fix loop:** (1) compile `.tex` to PDF via `tectonic` (or
  `pdflatex`), fixing compilation errors; (2) read the PDF and check for broken
  figures, unresolved cross-references, overflow, different-height composite
  panels, unreadable text, cut-off content, awkward page breaks; (3) fix `.tex`,
  recompile, re-read; (4) iterate until passing (max 3 iterations before
  flagging to the orchestrator). A single compile-and-ship pass is not
  acceptable. The final PDF is the deliverable, not the pandoc output.

**Typesetting does NOT modify physics content** — only layout, formatting, and
figure grouping. Never changes numbers, captions (except grammar), or section
structure. If a physics issue surfaces (missing figure, inconsistent number),
fix it at the source in the AN prose, not in the `.tex`.

See `04-output.md` for the full AN specification.

**Artifact:** `ANALYSIS_NOTE_5_v1.md` + compiled PDF + `results/` directory.
**Review:** 2-bot (§6).

---

## 3a. Orchestration and Agent Architecture

---

### 3a.1 Orchestrator Architecture

The orchestrator is a **thin coordinator** — it spawns subagents, reads
summaries, makes phase-transition decisions, and commits. It never writes
analysis code, produces figures, or debugs. Subagent contexts are discarded
after each phase; the orchestrator stays small.

**The orchestrator loop** is EXECUTE → REVIEW → CHECK → COMMIT → ADVANCE. The
canonical loop (with human gates, anti-patterns) lives in
`templates/root_claude.md` (auto-loaded at runtime). This section gives
architectural rationale; the template gives operational instructions.

**Review is always by subagent.** Self-review is acceptable only for Phase 2.

**Anti-patterns:** Skipping phases. Writing code as orchestrator. Accepting weak
reviews to save tokens. Spawning subagents without `model: "opus"`.

**Binding commitment tracking.** The orchestrator maintains awareness of all
"Will implement" commitments from the Phase 1 strategy. At each gate, verify all
commitments scheduled for that phase are fulfilled. Unfulfilled binding
commitments are Category A regardless of whether the reviewer catches them — the
orchestrator is the last line of defense. A deferred commitment (e.g. generator
comparison 4a→4b) must be explicitly documented with justification and a hard
deadline. Commitments cannot be deferred indefinitely.

---

### 3a.2 Subagent Roles and Context

Subagents are **executors** or **reviewers**. Each receives curated context
(§3a.4). See `appendix-prompts.md` for literal prompt templates.

**Executors** receive phase CLAUDE.md, upstream artifacts, experiment log,
conventions. They work plan-then-code: `plan.md` first, then code in `src/`,
figures in `outputs/figures/`, artifact last.

**Reviewers:**

| Role | Context | Goal |
|------|---------|------|
| Physics reviewer | Physics prompt + artifact only | "Would I approve this for publication?" |
| Critical reviewer | Full context + conventions + RAG corpus | Find all flaws in correctness and completeness |
| Arbiter | All reviews + artifact + conventions | PASS / ITERATE / ESCALATE |

**Reviewer RAG access.** The critical reviewer has experiment-corpus access (MCP
tools) and should query it to verify claims, check how reference analyses
handled similar concerns, and identify published standards before accepting or
rejecting a questionable approach (flat systematic, novel validation criterion).

**2-bot review:** the physics reviewer runs first (parallel with the plot
validator at figure phases), then the arbiter. See §6.2–6.4. The bar is high:
ITERATE liberally.

---

### 3a.3 Health Monitoring

- **Commit before spawning** each subagent (checkpoint).
- **Monitor agent progress.** If an agent produces no output/commits for a
  sustained period, check logs before respawning.
- **Context splitting** for Phase 4b/5: separate subagents for statistical
  analysis and AN writing.
- **Phase 4/5 execution pipeline.** At sub-phases that produce/update the AN
  (4a, 4b, 4c, 5), the executor performs statistical analysis, AN writing, and
  typesetting in a single role. The PDF must exist before review at 4a and 4b;
  the review panel reads the PDF. Under heavy context pressure the orchestrator
  may split statistical analysis and AN-writing/typesetting into separate
  executor invocations (the second reading the inference artifact from disk) — a
  judgment call.
- **Session logs survive crashes.** Every agent writes an incremental session
  log to `logs/` (see `appendix-sessions.md`). After a stalled agent is killed,
  check its session log to understand what was accomplished before respawning.

---

### 3a.4 Context Management

**Artifacts are the only handoff.** No conversation history, no shared
variables. Each session starts from artifacts + instructions.

**Three context layers per agent:**

1. **Bird's-eye framing (~1 page):** physics prompt, analysis type, current
   phase, applicable conventions, end goal (publication-quality AN).
2. **Relevant methodology sections (~2-5 pages):**

| Role | Sections |
|------|----------|
| Phase 1 executor | §1, §2, §3 (Phase 1), §5, §7 |
| Phase 2 executor | §3 (Phase 2), §5, §7, Appendix D |
| Phase 3 executor | §3 (Phase 3), §5, §7, §11, Appendix D |
| Phase 4 executor | §3 (Phase 4), §4, §5, §7, §11, Appendix D |
| Phase 5 executor | §3 (Phase 5), §5, Appendix D |
| 2-bot reviewer | §6, applicable phase from §3, conventions, checklist |
| 1-bot reviewer | §6, applicable phase from §3, conventions |
| Arbiter | §6, conventions |

3. **Upstream artifacts (~2-10 pages):** prior phase artifacts + experiment log
   if continuing within a phase.

**Context budget:** ~20-30 pages max at Phase 5. Summarize artifacts exceeding
~5 pages. Experiment log consulted on demand, not loaded in full.

**Artifacts before speed.** When context pressure mounts, write the current
artifact and stop cleanly. Never skip an artifact to "save context."

---

### 3a.5 Parallelism and Sub-delegation

- **Within a phase:** parallel sub-agents writing to separate directories,
  consolidated before review.
- **Across phases:** sequential (Phase N reads Phase N-1 artifact).
- **Per-channel:** channel-specific work in Phases 2-3 can run in parallel.

**Sub-delegation within a phase:** delegate compute-heavy tasks (MVA training,
systematic evaluation, plot generation, closure tests) to sub-agents. The
executor coordinates and integrates, retaining judgment (which backgrounds,
whether closure tests pass).

---

## 5. Artifact Format

### 5.1 Experiment Log

Each phase maintains `experiment_log.md` — an append-only lab notebook of what
was tried and what happened. Never deleted or modified. An empty log at phase
end is a review finding. Append after every material decision, discovery, or
failed attempt. The formal artifact references the log for alternatives
explored; agents read the log on demand to avoid re-trying failed approaches.

### 5.2 Primary Artifact

Every phase produces a markdown artifact — the handoff to subsequent phases and
the permanent record. Artifacts must be **self-contained**: a reader with only
the artifact and experiment corpus understands what was done and why.

**Artifacts are AN source material.** Phase 4 artifacts must be at publication
quality — the Phase 4b/5 agent reads them to draft the AN. Terse artifacts
produce terse AN sections.

**AN versioning.** The analysis note is phase-stamped and never overwritten:
`ANALYSIS_NOTE_{phase}_v{N}.{md,tex,pdf}`. Each phase (4a, 4b, 4c, 5) produces a
new phase-stamped v1; review/fix cycles within a phase increment the version
(v2, v3, …). All versions preserved on disk. Each phase's executor reads the
latest version from the previous phase and produces a new phase-stamped v1.

**Standard sections:** (1) Summary — what was accomplished (1 paragraph); (2)
Method — reproducible detail; (3) Results — tables, figures (by path), numbers
with uncertainties; (4) Validation — checks performed, quantitative outcomes;
(5) Open issues — what subsequent phases should know; (6) Code reference —
`pixi run <task>` commands that produced results.

**Figure captions must be self-contained**, format `<Plot name>. <Context and
conclusion.>`, 2-4 sentences: name the plot, state context not visible (selection,
normalization, what it validates), give the key conclusion. **Do not restate the
legend or axis labels** — the caption adds what the plot cannot convey. Under two
full sentences is Category A. See `appendix-plotting.md` § "Captions".

**Caption co-generation.** Every figure has a draft caption written when the
figure is produced (same script or first-referencing artifact). The Phase 5 AN
writer refines them but never writes captions from scratch for figures it didn't
produce.

**Flagship figures.** Phase 1 defines ~6 "money" figures, produced at highest
quality in Phase 5 with extra review attention (tighter axis limits, careful
legends, considered colors). The list propagates from strategy to the AN writer.
Every flagship figure appears in Results or Comparison, not buried in an appendix.

**Dead-end approaches.** The experiment log records all approaches tried,
including failures; the AN presents the variable survey, selection rationale,
final method, and result. Any approach seriously attempted and rejected (BDT
tagger rejected for data/MC disagreement, alternative unfolding that failed
closure) must be documented with **evidence** in an appendix subsection: (1)
what was tried (1-2 sentences); (2) why it failed, with a quantitative criterion
("data/MC chi2/ndf > 5 on 3 of 13 inputs"); (3) ≥1 diagnostic figure showing the
failure mode (produced and saved during the rejecting phase); (4) what is used
instead and why it's better. "Rejected due to data/MC disagreement" without the
figure is Category B. Purely exploratory dead ends (a cut value with worse S/B)
need only a log sentence. Threshold: if the approach involved writing and
running code, its failure deserves a figure.

**Numerical self-consistency.** Every value in the AN must appear consistently
everywhere quoted (per-section tables, summary tables, prose, derived-quantity
calculations, appendix tables). A fix cycle that changes a result updates ALL
instances. The `results/*.json` files are the single source of truth; the AN
renders them into prose. A per-section table contradicting the summary table is
Category A regardless of which is correct — the inconsistency itself is the
problem. See the fixer's PROPAGATE step and the executor's NUMBERS CONSISTENCY
LINT.

**Supplementary files** (`.npz`, `.json`, workspaces, trained models) include a
brief artifact description: what the file contains, how to load it, which pixi
task produced it.

**Supplementary artifacts:** `UPSTREAM_FEEDBACK.md` (non-blocking feedback to an
earlier phase) and `REGRESSION_TICKET.md` (regression investigation output).
See §6.7.

---
