## 6. Review Protocol

### 6.1 Classification
- **(A) Must resolve** — a genuine correctness error: a clearly wrong result, broken computation, circular/tautological method, or a result contradicting a well-measured reference (§6.8). Only Category A blocks advancement.
- **(B)/(C) Advisory** — anything that is not a correctness error (completeness gaps, prose, style, "could be stronger"). Optional notes; they do NOT block PASS.

### 6.2 Review gate
Exactly **one reviewer** per gate — the critical reviewer (`agents/critical_reviewer.md`) — except Phase 2 (self-review). No panels, no separate adjudicator: the reviewer reads the artifact, checks correctness, and issues PASS / ITERATE. (Analyses may locally reduce which phases get a reviewer; the orchestrator regression checklist then covers the rest.) Default: reviewer at 1, 3, 4a, 4b→human gate, 4c, 5. Category A → fixer → re-submit.

### 6.3 Reviewer focus
Narrow: **are the results correct, reproducible, and is anything clearly broken?** Block only on genuine correctness errors; record everything else as advisory. The standing correctness gates are the closure alarm bands, the fit-triviality/circularity gate, the tautological-comparison gate, and §6.8.
- **Evidence-based:** a verification claim cites a specific number/file ("closure chi2/ndf = 1.3/36, p=0.24 from results/closure.json"). "Looks reasonable" with no evidence does not clear a concern.
- **Investigation subagents:** when a concern needs tracing through code, the reviewer may spawn a read-only subagent (specific question + paths; it reports, never fixes) and cite its findings.

### 6.4 Focus by phase
- **Ph1:** no correctness gates yet (sanity-check coherence). **Ph2:** self-review.
- **Ph3:** closure alarm bands (Category A): chi2/ndf < 0.1 (suspiciously good — inflation/tautology); chi2/ndf > 3 or any pull > 5σ (method failure); `passes:false` in machine output while text claims acceptable (misrepresentation).
- **Ph4a:** closure bands; circular/tautological fitted parameters; §6.8. A systematic with exactly-zero impact in every bin is likely broken — verify the varied input is non-trivial.
- **Ph4b:** AN numbers match the latest machine outputs (stale = correctness error); compatible with 4a; §6.8.
- **Ph4c:** GoF of the primary config (chi2/ndf < 3, p > 0.01); fit-triviality gate (chi2 ≈ 0 / circular inputs); fit pathologies investigated not worked around; §6.8 vs every reference; AN numbers match JSON.
- **Completeness (Ph1, 4a):** the 4a systematic completeness table (`| Source | Conventions | Ref1 | Ref2 | This | Status |`) — gaps are advisory unless a committed source is silently dropped.
- **Figures:** Category-A red flags — data/MC normalization off >~20% across the bulk; postfit worse than prefit; empty bins where events expected; ratio globally off 1.0; efficiency/purity outside [0,1]; negative yields; distributions that should differ appearing identical; error bars larger than the signal / spanning the y-axis (often missing explicit `yerr`). Illegible text, legend overlap, axis ranges are advisory.
- **Tautological comparisons (Category A):** if "comparison to theory/MC" is mathematically identical to "comparison to expected" (expected derived from the same MC), do not present them as independent. Alarms: chi2/ndf < 0.1 between corrected data and the MC it was corrected with; chi2 identically zero (circularity); per-subset fits returning identical central values. True validation needs an independent dataset, a different published analysis, a different method on the same data, or theory not used in the chain.

### 6.5 Iteration and scoping
Reviewer → fixer loop until every Category A clears; escalate to the human if stuck after a few tries. The reviewer must not dismiss a cheap-to-fix correctness finding as "out of scope" (re-running a Phase-4 script, adding a figure, propagating a systematic is in scope). Batch findings that need an earlier-phase re-run into one regression iteration (§6.7).

### 6.6 Human gate
Between 4b and 4c. The human receives a compiled PDF (not markdown) with 10% results + the unblinding checklist.

| Response | Meaning | Action |
|---|---|---|
| APPROVE | sound; proceed | Phase 4c |
| ITERATE | issue within 4b scope | fix, re-review 4b, re-present |
| REGRESS(N) | fundamental issue from Phase N | non-destructive regression (below) |
| PAUSE | external input needed | wait |

APPROVE means the methodology and 10% validation are clean — NOT the final result (that follows Phase 5). The agent never fully unblinds autonomously. The human checklist covers: selection yields match the reference within ~10%; key data/MC acceptable; closure/stress tests pass; systematic table covers the reference sources; 10%/expected agrees with the reference within uncertainties; figures interpretable; and especially **"where I wasn't sure" items** — the agent flags low-confidence physics judgments for explicit human review.

**Non-destructive regression:** (1) Investigator assesses cascade scope; (2) writes `REGRESSION_TICKET.md` (what changes in Phase N; which downstream artifacts are invalidated vs reusable); (3) executor creates **new artifact versions** (e.g. `STRATEGY_v2.md`), never overwrites; (4) each downstream phase reuses if unaffected, re-executes if invalidated; (5) executor produces a new AN version; (6) affected phases re-reviewed; (7) re-present. **Post-regression AN cohesion:** the body reads as if the current approach was always the plan; the Change Log carries the audit trail.

### 6.7 Phase regression
**Triggered, not avoided.** Concrete triggers (must not be rationalized away): data/MC disagreement on observable/MVA inputs; closure failure (p < 0.05); stress-test failure without successful remediation; operating-point instability; single systematic > 80% of total; MC used for periods without simulation; result > 3σ or > 30% from a well-measured reference (§6.8); GoF toy distribution inconsistent with observed chi2 (outside the 95% interval); > 50% of bins excluded by a flat-prior/similar gate; two supposedly-independent distributions visually identical; systematic double-counting (identical per-bin shifts under different names); correlated-combination error (shared sources treated as uncorrelated).

**Procedure:** document the issue + origin phase → classify as a trigger in the review artifact → orchestrator spawns the **Investigator** (→ `REGRESSION_TICKET.md`) → fix origin → re-review → re-run affected downstream.

**Upstream improvement cascade (all fixes):** when any component improves, trace every downstream consumer (script/figure/table/AN section), re-run or document why unaffected; if a downstream result shifts > 1σ or changes a ranking, propagate further. Re-running is cheap; stale contradictory results are expensive — when in doubt, re-run. Presentation issues (labels/captions) are Phase-5 iteration, not regression. Any executor may file a non-blocking `UPSTREAM_FEEDBACK.md`.

### 6.8 Validation target rule
Applies at the 4a/4b/4c/5 gates (not Ph1; not Ph2–3).

**Tier 1 — mandatory investigation (Category A).** Any extracted parameter with **pull > 3σ from a well-measured reference OR relative deviation > 30%** blocks advancement until resolved. (30% is guidance to trigger investigation, not gospel — use physics judgment.)

**Tier 1b — method-improvement (Category B, upgradable to A).** For a 1–3σ (or 10–30%) deviation with a known directional bias from method choice (NLO vs NNLO, mean vs differential fit), check: is a better method feasible with available data/tools? did the reference analyses use it? would it plausibly resolve the deviation? If all yes, require the better method as primary (or a documented cross-check). Using a simpler method every published analysis avoided is a method-parity failure. "A known limitation of our method" is not sufficient when the better method is feasible.

**Tier 2 — calibration-first.** A method that passes MC closure but deviates on data is the classic calibration-mismatch signature. (1) **Diagnose:** back-substitute observed rates + the known reference to get implied MC-derived parameters; compare to nominal. Back-substitution is a diagnostic, **not** a calibration — using it as the correction is circular and forbidden as primary. (2) **Plausibility:** are implied shifts consistent with known data/MC differences? A 5% shift matching a 5% resolution diff is plausible; a 50% shift with no mechanism is a red flag. (3) **Calibrate from independent observables**, stating independence level — *Independent* (gold: control sample / tag-and-probe / sideband with no dependence on the primary observable; data/MC ratio = scale factor), *Weakly dependent* (acceptable with the residual circularity quantified), *Circular* (forbidden as primary — diagnostics/systematic-range only). (4) Parameters that can't be independently calibrated: use the MC value as central and inflate the systematic to cover the MC-to-back-substituted range. (5) **Magnitude closure:** independent calibrations + inflated systematics must account for the full deviation (uncalibrated result within ~2σ), else a bias source is missing. (6) Document the full chain in the AN. If the deviation can't be explained by calibration: check bugs/signs/units/inputs; if none, document as a genuine tension and consider downscoping the parameter — do not claim new physics without exhausting mundane explanations.

---

## Appendix B: Minimal Artifact Checklist

| Phase | Artifact | Must contain |
|---|---|---|
| Strategy | `STRATEGY.md` | signal/background enumeration, selection approach, blinding plan, systematics categories, citations, reference-analysis systematic table |
| Exploration | `EXPLORATION.md` | sample inventory, data quality, variable ranking, preselection cutflow, data/MC comparisons |
| Processing | `SELECTION.md` | selection + region definitions, background estimates, closure tests, per-cut distributions |
| 4a | `INFERENCE_EXPECTED.md` | per-source systematic table, fit/correction model, expected results, validation tests, completeness table, covariance |
| 4b | `INFERENCE_PARTIAL.md` + `ANALYSIS_NOTE_4b_v1.md` | 10% results, post-fit diagnostics, GoF, draft AN |
| 4c | `INFERENCE_OBSERVED.md` | full results, post-fit diagnostics, anomaly assessment, comparison to expected |
| Documentation | `ANALYSIS_NOTE_5_v{final}.md` + `results/` | complete AN, figures, math, machine-readable results |

**AN completeness (each absent item is Category A):** LaTeX math throughout; one subsection per systematic and per cross-check; per-cut distributions + efficiencies; MVA diagnostics when used; data/MC for every selection variable; fit diagnostics (NP pulls, GoF, post-fit); full covariance in appendix; machine-readable `results/`; quantitative comparison to published data; `pixi.toml` `all` task reproduces the chain; non-empty experiment log; all phase artifacts on disk; no empty sections (≥1 prose paragraph before any figure/table); numerical self-consistency (every prose/table/section number matches the summary and `results/*.json`). Experiment log must record data-format discovery, key parameter choices, failed approaches, and bugs + resolutions.
