## 3. Analysis Phases

Five sequential phases; parallelize independent tasks within a phase (§3a.5).

### 3.0 Gates
Every phase boundary is a hard gate: Phase N+1 cannot begin until Phase N produced its artifact AND cleared its gate — the critical reviewer at Phases 1/3/4a, the human gate at 4b, and the orchestrator's regression checklist at the unreviewed phases (4c, 5); see §6.2. Protocol: produce artifact → update experiment log → review → resolve Category A → advance. Never skip an artifact, even under context pressure — write it and stop cleanly. Phase CLAUDE.md templates (from `templates/`) are the runtime entry points.

### Analysis types
**Search:** signal/background, SR/CR structure, blinding (§4), limits/significance. **Measurement:** corrected spectra / extracted parameters, staged validation (§4) replaces blinding. Where a phase references search concepts (SR, CR, S/B), measurements substitute fiducial region / sidebands / purity.

### Philosophy
- **Correctness above all.** The right answer with honest uncertainties; no polish compensates for a wrong result. Investigate unexpected validation results until you understand WHY; assume you are wrong when disagreeing with a published value until proven otherwise.
- **Never adjust parameters to match.** Every parameter needs a justification fixed BEFORE seeing its effect. Tuning a cut until data/MC agrees, dropping a systematic for being "too large", smoothing a band, or choosing binning that hides a discrepancy is fabrication — the #1 failure mode (training rewards agreeable outputs). The goal is the most convincing physics, not passing gates.
- **The "nodding physicist" test.** At every phase (incl. 4a), would a physicist find every step well-motivated, every number supported, every limitation honestly assessed with evidence that improvement was attempted?.
- **Solve problems, don't accept limitations.** Poor tagger purity → alternative variables / contamination correction; dominant systematic → better evaluation method. Accept a limitation only when (a) a concrete approach was tried, (b) it failed for a specific documented reason, (c) solving needs compute over a few minutes or unavailable resources. A limitation without evidence of attempted improvement is Category B.
- **Every measurement needs context** (compare to reference, state consistency + why, what question it answers, what it can resolve) and **cross-checks are the physics content** (subperiod stability, published overlay with chi2). **Published-overlay is mandatory at 4a** (deferring to Phase 5 is Category B). 

---

### Phase 1: Strategy
Written strategy a collaboration reviewer could approve. The agent must:
- Query the corpus for  prior work, datasets; identify signal, backgrounds (irreducible/reducible/instrumental), discriminating variables.
- **Technique justification** against alternatives; **selection-approach exploration plan** — identify ≥1 candidate approaches
- **Method parity (binding):** compare the proposed method — including the statistical extraction — to reference publications; if they used a more sophisticated method, commit to it/better or justify a simpler one by a specific technical limitation AND implement the published method as a cross-check. "Easier" is not a justification. Reviewed at 4a; silent downscoping is Category A.
- **Systematic plan:** read the applicable `conventions/` doc; enumerate every required source "Will implement"/"Not applicable because…" (binding, reviewed at 4a).
- Identify 2 reference analyses, tabulate their systematic programs, and **extract their published numerical results** into the artifact (binding §6.8 comparison targets at 4c — do not defer to Phase 5).
- Label constraints [A], limitations [L], decisions [D] (propagate to later phases + AN).
- **Measurements:** define observable(s), correction strategy, prior measurements (validation target), theory predictions. **Theory-comparison independence:** comparison generators must include ≥1 independent of the MC used for the correction (else a closure check, not a theory comparison). Identify ~6 flagship "money plots" (produced at top quality in Phase 5) and any methodology diagrams.

**Artifact:** `STRATEGY.md`. **Review:** 1 reviewer.

### Phase 2: Exploration
Characterize data, validate the detector model, establish the selection foundation. The agent must:
- Inventory samples (files/trees/branches/events/cross-sections); validate data quality; apply standard object definitions; survey discriminating variables ranked by separation; **data/MC chi²/ndf on every candidate variable** (feeds Phase 3's variable quality gate); establish baseline yields after preselection.
- **Published yield cross-check** when data is pre-selected: N_exp = L_pub × σ_pub; f_presel = N_obs/N_exp per energy point/year; flag energy dependence > 2% (biases shape measurements — a Phase-4 input).
- **Data discovery:** metadata → ~1000-event slice → identify jagged structure → document schema.
- **Data archaeology (archived/open data):** (1) check all weight/flag branches for non-trivial values; (2) compare counts to σ×L to detect pre-selection and what was cut; (3) check MC generator/tune/energy coverage; (4) check truth-level info. (5) **Strategy-revision gate:** if a discovery materially changes feasibility, flag the artifact as a strategy-revision input ("Phase 1 assumed X; Phase 2 found Y; implication Z"); the orchestrator updates STRATEGY.md before Phase 3 (normal flow, lightweight re-review of changed sections — not a regression).
- **PDF build test:** stub `pixi run build-pdf` to verify the toolchain (can run in parallel).

**Artifact:** `EXPLORATION.md`. **Review:** self-review.

### Phase 3: Processing
Implement the strategy (don't redesign it). Searches: selection, regions, background estimation. Measurements: selection, correction chain.
- **Approach comparison (mandatory):** try ≥2 selection approaches, compare quantitatively with a common FoM on the same sample, document why the chosen one wins (exemption only if Phase 1 documented alternatives infeasible and the review validated it).
- **MVA input variable quality gate:** survey table (variable | discrimination | data/MC chi²/ndf | decision). Inputs with data/MC chi²/ndf > 5 are discarded unless exceptional discrimination AND a validated data-driven calibration (a poorly-modelled input biases the data result even when MC closure passes). When most inputs fail, the MVA may still be viable if the BDT-output data/MC disagreement is smaller and calibrated with a control-region scale factor + systematic (standard b-tagging practice — attempt before rejecting). If MVA: train + ≥1 alternative architecture, multiclass if >2 classes, check data/MC on the classifier output, save ROC/score/importance for the AN.
- Every cut motivated by a plot (N-1 preferred); cutflows monotonically non-increasing.
- **Searches:** define CRs (background-enriched) and VRs (independent, between CR and SR); estimate SR from CRs; VR closure p > 0.05 or Category A.
- **Measurements:** data/MC for all observable-entering variables; response matrix (diagonal fraction, condition number) — **early gate:** compute it on ~10K MC events, if diagonal < 50% investigate before the full chain; closure + stress tests (failure p < 0.05 is Category A); justify binning.
- **Validation-failure remediation:** a failed required test needs **≥3 independent remediation attempts** (≥1 from a minimal-context subagent) before being documented as a limitation; read how published analyses solved it. Accepting a failure without remediation is Category A.
- **Wholesale bin exclusion** (>50% of bins by a flat-prior/similar gate) is a red flag → re-evaluate binning/method.

**Closure alarm bands (Phase 3 AND 4a, non-negotiable):** chi²/ndf < 0.1 → Category A (suspiciously good — inflation/tautology/same-sample); chi²/ndf > 3 or any pull > 5σ → Category A (method failure: fix the bug, redesign, fix the test, or formally abandon); closure `passes:false` while text claims acceptable → Category A. The first hypothesis for a failure is always a code bug, not physics.

**Artifact:** `SELECTION.md`. **Review:** 1 reviewer.

### Phase 4: Statistical Analysis
Three sub-phases; both measurements and searches follow 4a → 4b → 4c.

#### Phase 4a — Expected results (Asimov only)
- **Published-input lookup:** when the strategy commits to a published value [D] (e.g. luminosity), obtain the actual published value — escalate RAG → `get_paper` → fetch the PDF → orchestrator blocker; substituting a data-derived value is Category A.
- **Variation sizing:** every systematic motivated by a measurement/calibration/published uncertainty, not a round number ("±50% on background" is Category A unless 50% IS the measured uncertainty). 
- **Extraction method hierarchy** (parameter measurements): differential fit with full covariance (default) > moment fit > total-rate/mean (cross-check only). When a corrected differential distribution AND covariance exist, mean-value-only is a downscope that must be justified; implement both with the differential fit primary.
- Construct the binned likelihood (Asimov data + systematic NPs); confirm the fit converges and results are sensible. **Validation (lean scope):** validation at 4a is applied ONLY to the fit of the final distribution used for signal extraction — confirm convergence on the correct Asimov and recovery of the injected inputs, with GoF (chi²/ndf) reported. Broad batteries (signal-injection linearity scans, ≥5-quantity pull distributions, toy saturated-model p-values) are NOT required at 4a in this lean scope. **Fit boundary check:** no fitted parameter within 1% of a boundary (else the uncertainty is a lower bound — widen/refit or document saturation).
- **Per-systematic documentation** in running prose (origin, evaluation method, numerical impact, interpretation — not bold-label form headings). **Phase-1 traceability:** re-read STRATEGY.md, every committed source implemented or formally downscoped [D] (silent absence is Category A).
- **COMMITMENTS.md:** list every Phase-1 commitment with machine-readable status (`[x]` resolved / `[D]` downscoped-with-justification / `[ ]` open); update at every boundary; any `[ ]` at Phase 5 is Category A.

**Artifact:** `INFERENCE_EXPECTED.md` + `ANALYSIS_NOTE_4a_v1.md` (complete AN, expected-only numbers). **PDF compilation mandatory** before review (markdown → pandoc `.tex` → `postprocess_tex.py` → figure composition → tectonic; the reviewer reads the PDF). **Number-consistency gate:** in this lean scope the gate does NOT run at 4a; it runs at the 4b unblinding, restricted to the final distribution used for signal extraction (see §4b). **Review:** 1 reviewer, **light check only** — confirm the fit converges on the correct Asimov and recovers the injected inputs (μ, m_H); no detailed/exhaustive review at 4a.

#### Phase 4b — 10% data validation
10% data (fixed seed), MC normalized to 10% luminosity; run the full chain; GoF, NP pulls, impact ranking; compare to 4a expected (overlay + chi2); for extraction include diagnostics sensitive to data/MC differences (not just the final quantity). Update the AN with 10% numbers. **Number-consistency gate (first applied here):** the final distribution used for signal extraction — its bin contents/yields and the extracted μ/m_H quoted in the AN must match the latest `results/*.json` (>1% = Category A). **Verify all figure references resolve before compiling** (a missing figure is Category A). **PDF mandatory** (the human gate reads the PDF). **Artifact:** `INFERENCE_PARTIAL.md` + `ANALYSIS_NOTE_4b_v1.md`. **Review:** **human gate** (no reviewer in the lean scope; §4.2) — the orchestrator runs its regression checklist, then presents the compiled PDF to the human.

#### Phase 4c — Full data
Full chain + post-fit diagnostics; compare to **both** 10% and expected (flag >2σ disagreement with expected). Investigate anomalies (large pulls, poor GoF). **Re-evaluate scan-based systematics on full data** (if data differs from MC by >2×, document which is used — transferring the whole MC budget unvalidated is a borrowed flat systematic). **Configuration selection must include GoF** — a config with chi²/ndf > 3 must not be primary without investigating the GoF and comparing to acceptable-GoF configs (selecting solely for small error is Category A). **Fit pathologies** (degeneracies, near-singular Hessian, boundary hits) investigated, not silently worked around. **Fit-triviality gate:** chi² ≡ 0, or fitted parameters exactly equal to chain inputs, → STOP and check algebraic circularity (trace every input to the cross-section formula; if any was derived from the same theory cross-section the fit measures, it is circular — seek independent published inputs; only then proceed, presenting values as a "self-consistency check", not a measurement). **Poor GoF after a new correction** → first hypothesis is a missing calibration (examine per-point residuals, calibrate, refit) — do not revert to the circular approach. **Viability check** for every reported result (§6.8): >3σ pull or >50% relative deviation without quantitative explanation is unacceptable; unphysical intermediate values → "not reliably extractable". **Machine-readable `results/`** JSON is the single source of truth the AN reads from. **AN + PDF update mandatory. Review:** none in the lean scope — covered by the orchestrator's regression checklist (incl. the §6.8 validation-target check).

### Phase 5: Documentation
Final analysis note — publication-quality, self-contained. The executor does figures → prose → PDF in one role.
- **Figures:** produce remaining AN-specific figures (per-cut, per-systematic); flagship figures get extra care; produce Phase-1 methodology diagrams.
- **AN writing:** read all phase artifacts + figures; write the complete AN; completeness test = a physicist reproduces every number from the AN alone. Interpretive quality: **≥4 equations** (observable, correction, systematic, fit — zero is Category A); 2–3 interpretive sentences after each results table/key figure; a **validation summary table**; a **resolving-power statement**; **≥1 published-overlay figure with a chi2**.
- **Number-consistency gate** before compiling (>1% vs `results/` JSON = Category A).
- **Figure composition annotations** (`<!-- COMPOSE: NxM grid -->`, `<!-- COMPOSE: side-by-side -->`, `<!-- FLAGSHIP -->`) mark related figures for merging at typesetting (a physics judgment, persists across versions).
- **Typesetting:** pandoc markdown → `.tex` (`--standalone --number-sections --toc --filter pandoc-crossref --citeproc`, include `preamble.tex`) → `postprocess_tex.py` → executor merges annotated figure groups into side-by-side `\includegraphics`+`\hspace` composites (no `\subfloat`; unified (a)/(b) captions; merging per-variable/per-systematic/per-cut runs and nominal+uncertainty pairs is mandatory — Category A if left standalone), converts longtables, verifies prose-before-figures and caption quality → **compile → read → fix loop** (tectonic; max 3 iterations; the PDF is the deliverable, not the pandoc output). Typesetting changes layout only, never numbers/structure — fix physics issues at the source.

**Artifact:** `ANALYSIS_NOTE_5_v1.md` + compiled PDF + `results/`. **Review:** none in the lean scope — covered by the orchestrator's regression + AN-completeness checklist (a fixer addresses any Category A it surfaces). See `04-output.md` for the full AN spec.

---

## 3a. Orchestration and Agent Architecture

### 3a.1 Orchestrator architecture
A **thin coordinator** — spawns subagents, reads summaries, makes phase-transition decisions. Never writes analysis code, produces figures, or debugs; subagent contexts are discarded after each phase so the orchestrator stays small. Loop: EXECUTE → REVIEW → CHECK → COMMIT → ADVANCE (canonical version in `templates/root_claude.md`). Self-review only at Phase 2. **Anti-patterns:** skipping phases; writing code as orchestrator; accepting weak reviews to save tokens; spawning subagents without `model: "opus"`. **Binding-commitment tracking:** at each gate verify all Phase-1 "Will implement" commitments scheduled for that phase are fulfilled — unfulfilled ones are Category A regardless of whether the reviewer caught them (the orchestrator is the last line of defense).

### 3a.2 Subagent roles
**Executors** receive phase CLAUDE.md + upstream artifacts + experiment log + conventions; work plan-then-code (`plan.md` first, code in `src/`, figures in `outputs/figures/`, artifact last; from 4a on they also write + typeset the AN). The **critical reviewer** (single reviewer per gate, full context + conventions + RAG) finds correctness errors and issues PASS/ITERATE directly; it queries the corpus to verify claims and check published standards before accepting a questionable approach. See `06-appendix.md` for literal prompt templates.

### 3a.3 Health monitoring
Commit before spawning each subagent (checkpoint). Monitor progress; check the session log (in `logs/`, crash-resilient) before respawning a stalled agent. For Phase 4b/5 under heavy context pressure, the orchestrator may split statistical analysis and AN-writing/typesetting into separate executor invocations (the second reading the inference artifact from disk). The PDF must exist before review at 4a/4b.

### 3a.4 Context management
**Artifacts are the only handoff** — no conversation history, no shared variables; each session starts from artifacts + instructions. Three layers per agent: (1) bird's-eye framing (~1 page: physics prompt, analysis type, current phase, conventions, end goal); (2) relevant methodology sections (~2–5 pages, per role — e.g. Phase-4 executor: §3 Phase 4, §4, §5, §7, §11, plotting; reviewer: §6 + the phase + conventions + checklist); (3) upstream artifacts (~2–10 pages). Budget 5-10 pages; summarize artifacts over ~5 pages; consult the experiment log on demand. When context pressure mounts, write the artifact and stop cleanly — never skip an artifact to save context.

### 3a.5 Parallelism
Within a phase: parallel subagents writing to separate directories, consolidated before review. Across phases: sequential. Per-channel work in Phases 2–3 can run in parallel. Delegate compute-heavy tasks (MVA training, systematic evaluation, plotting, closure tests); the executor coordinates and retains physics judgment.

---

## 5. Artifact Format

### 5.1 Experiment log
`experiment_log.md` is an append-only lab notebook (never deleted/modified) of what was tried and what happened. An empty log at phase end is a review finding. Append after every material decision, discovery, or failed attempt; agents read it on demand to avoid re-trying failed approaches.

### 5.2 Primary artifact
Every phase produces a self-contained markdown artifact (the handoff + permanent record); a reader with only the artifact + corpus understands what was done and why. Phase-4 artifacts must be at publication quality (terse artifacts → terse AN sections). **AN versioning:** phase-stamped, never overwritten — `ANALYSIS_NOTE_{phase}_v{N}.{md,tex,pdf}`; each phase reads the latest prior version and produces a new phase-stamped v1; fix cycles increment v. **Standard sections:** Summary, Method (reproducible), Results (tables/figures/numbers+uncertainties), Validation (quantitative outcomes), Open issues, Code reference (`pixi run <task>`).
- **Figure captions** self-contained, `<Plot name>. <context + conclusion.>`, 2–4 sentences (not a restatement of axes/legend); under two sentences is Category A. Captions are co-generated when the figure is produced; the Phase-5 writer refines, never writes from scratch for figures it didn't produce.
- **Flagship figures** (Phase 1's ~6) appear in Results/Comparison, not an appendix.
- **Dead-end approaches** that involved writing+running code are documented in an appendix subsection with a quantitative failure criterion AND ≥1 diagnostic figure (criterion-only is Category B); purely exploratory dead ends need only a log sentence.
- **Numerical self-consistency:** `results/*.json` is the single source of truth; every AN instance (per-section tables, summary, prose, derived calcs, appendix) must match — a contradiction is Category A regardless of which is correct.
- **Supplementary files** (`.npz`/`.json`/workspaces/models) carry a brief description (contents, how to load, producing pixi task). Supplementary artifacts: `UPSTREAM_FEEDBACK.md`, `REGRESSION_TICKET.md` (§6.7).
