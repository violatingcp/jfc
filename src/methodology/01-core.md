# LLM-Driven HEP Analysis: Methodology Specification

This spec defines *what* each phase must produce, not *how*. The agent selects tools, writes code, and makes physics judgments within these constraints.

## 1. Scope and Principles

- **Quality bar: publication-ready.** Would a senior reviewer approve this for publication, not just "good enough to move on"?
- **Default posture: strongest achievable result.** Try the most powerful approach first; when the spec is silent, prefer the choice that makes the analysis hardest to criticize.
- **Artifacts over memory.** Each phase produces a self-contained report; later phases read reports, not conversation history.
- **Review at every gate** that has one; the agent adapts (omitting an unnecessary step is correct; performing one without justification is not).
- **Conventions over encoded physics** — operational knowledge lives in `conventions/`, consulted at Phases 1, 4a, 5.
- **Downscope as last resort** — and only after attempting the stronger method or documenting infeasibility (§12).

## 2. Inputs

**2.1 Physics prompt.** Natural-language goal: target and constraints (dataset, final state, energy). Need not specify methodology.

**2.2 Experiment context (RAG, when available).** Query a corpus (SciTreeRAG via MCP) for detector specs, object/MC definitions, performance numbers, prior analyses; cite all retrieved info. No RAG: proceed on `docs/` + training knowledge, mark uncorroborated claims "unverified" and flag. Data is ground truth — on discrepancy, trust data. Log failed retrievals to `retrieval_log.md`.

**2.3 Numeric constants and reference values.** **Never quote numeric constants from training data.** Every number entering the analysis — masses, widths, branching ratios, couplings, cross-sections, luminosities, beam energies, QCD/radiative coefficients, world averages, SM predictions, validation targets — must come from a citable source: RAG (paper ID + table/eq), web fetch (PDG live, HEPData, official), or a published paper (full ref). **At review, any uncited numeric constant is Category A.**

**2.4 Self-consistency for derived constants.** For every derived constant, substitute PDG world-average inputs and confirm the formula reproduces the known result within quoted precision (~1%). A larger discrepancy signals a wrong input or convention mismatch.

**2.5 External-input validation.** For each external input (published width, theory cross-section, borrowed efficiency): identify it, attempt a rough data-driven estimate of the same quantity, compare (consistent within ~2σ → use external as primary and document; else investigate). Report both when an external input dominates (>30% of total). Document infeasibility if no data-driven estimate is possible.

## 7. Tools and Paradigms

**7.1 Preferred tools (use these, not alternatives).** ROOT I/O → `uproot`; arrays → `awkward`/`numpy` (not pandas for event data); histograms → `hist`/`boost-histogram` (ND axes for systematics); stats → `pyhf`/`cabinetry` (binned), `zfit` (unbinned) — not RooFit/custom; MVA → `xgboost`/`scikit-learn`; plotting → `matplotlib`+`mplhep`; jets → `fastjet` (Durham `ee_genkt` p=1 for e+e−; anti-kt for pp); logging → `logging`+`rich` (no bare `print`); docs → `pandoc`≥3 + pdflatex (never LLM for conversion); deps → `pixi`. b-tagging tiered: cut-based cross-check (always), BDT default, NN only if justified.

**7.2 Paradigms.** Prototype on a ~1000-event slice, then scale. Columnar analysis — arrays + immutable boolean masks, no event loops. Workspace (pyhf JSON) written to disk and committed; each fit has a pixi task. Plots are evidence — every claim has a figure or table. Pin random seeds; record versions. MC normalization: weight = σ·L / Σw_gen. When MC mismatches data after proper scaling, add control regions with floating normalizations — do not hand-scale MC. Systematic naming `{source}Up`/`{source}Down`. No bin with < ~5 expected events.

**7.3 Scale-out.** Estimate before running (input size, per-event cost on a slice, memory). <2 min local; 2–15 min multicore (`ProcessPoolExecutor`); >15 min SLURM (`sbatch --wait` or `--array`).

**7.4 Multiprocessing safety.** With process pools + threaded libs (fastjet, MKL), set `multiprocessing.set_start_method("forkserver"|"spawn", force=True)` — default `fork` can return cached parent data (plausible but wrong). Validation: N independent parallel outputs must not be bit-identical.

**7.5 Calibration verification.** Every calibration needs a before/after comparison plot; the correction must visibly improve the relevant distribution (narrower residuals, better data/MC). If it worsens a distribution, the correction is wrong (e.g. double-correction). Critical for archival data with undocumented processing — never apply corrections without verifying their effect.
