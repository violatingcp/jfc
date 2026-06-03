# LLM-Driven HEP Analysis: Methodology Specification

## 1. Scope and Principles

This spec defines *what* each phase must produce, not *how*. The agent selects
tools, writes code, and makes physics judgments within these constraints.

**Quality bar: publication-ready.** Would a senior physicist on a review
committee approve this? Not "good enough to move on" — good enough to publish.

**Default posture: strongest achievable result.** Try the most powerful
approach first; attempt difficult methods before falling back. When the spec is
silent, prefer the choice that makes the analysis harder to criticize.
Downscoping (§12) is for genuine constraints — missing data, infeasible methods
— not for avoiding difficulty.

**Design principles:**
- **Artifacts over memory.** Each phase produces a self-contained report;
  later phases read reports, not conversation history.
- **Review at every level** — plans, code, results, writeup.
- **The agent adapts.** Omitting an unnecessary step is correct; performing one
  without justification is not.
- **Conventions over encoded physics** — operational knowledge in
  `conventions/`, consulted at Phases 1, 4a, 5.
- **Downscope as last resort, don't block** — but downscoping without first
  attempting the stronger method (or documenting infeasibility) is a process
  failure. See §12.

## 2. Inputs

### 2.1 Physics Prompt
Brief natural-language description of the physics goal: target and constraints
(dataset, final state, energy range). Need not specify methodology.

### 2.2 Experiment Context (When Available)
The agent **may** have a RAG corpus (SciTreeRAG) via MCP tools. Query for
detector specs, object/MC definitions, performance numbers, prior analyses;
cite all retrieved information. **No RAG:** proceed on training knowledge plus
any `docs/`, marking uncorroborated claims "unverified — based on training
knowledge" and flagging for human review. **Retrieve, then verify:** data is
ground truth — on discrepancy, trust data and document it. **Retrieval fails:**
log to `retrieval_log.md`, rephrase, else proceed and flag the gap.

### 2.3 Numeric Constants and Reference Values
**Never quote numeric constants from training data.** Every number entering the
analysis — PDG masses, widths, branching ratios, couplings, cross-sections,
luminosities, beam energies, QCD/radiative coefficients, world-average
measurements, SM predictions, validation targets — must come from a citable
source: (1) **RAG corpus** (cite paper ID + table/equation), (2) **web fetch**
(PDG live tables, HEPData, official source), or (3) **published paper** (full
reference). LLM training data is NOT a source. E.g. $M_Z = 91.1876$ GeV must
cite its origin, not be asserted from memory. **At review: any uncited numeric
constant is Category A** — the reviewer must verify validation targets were
fetched, not recalled.

### 2.4 Self-Consistency Check for Derived Constants
For every derived constant (e.g. $R_l^{\mathrm{EW}}$, QCD/radiative
coefficients): substitute PDG world-average values into the formula and confirm
it reproduces the known PDG result within quoted precision. A discrepancy
exceeding ~1% indicates a wrong input, convention mismatch, or approximation
level that doesn't match the analysis precision.

### 2.5 External Input Validation
When an external measurement is used as input (published widths, theory
cross-sections, borrowed efficiencies), the executor must attempt a data-driven
estimate of the same quantity as a cross-check, even rough.
1. **Identify** every external input: quantity, source, role in the chain.
2. **Estimate from data** where feasible (even order-of-magnitude).
3. **Compare.** If broadly consistent (within ~2σ), use the external input as
   primary and document the cross-check; if inconsistent, investigate before
   proceeding.
4. **Report both** when an external input dominates uncertainty (>30% of total).
5. **Document infeasibility** if no data-driven estimate is possible.

---

## 7. Tools and Paradigms

### 7.1 Preferred Tools

| Capability | Tool | Notes |
|-----------|------|-------|
| ROOT file I/O | uproot | `uproot.open()` to explore, `arrays()` to load. NOT PyROOT/ROOT macros. |
| Arrays | awkward-array, numpy | Columnar, no event loops. awkward for jagged, numpy for flat. NOT pandas for event data. |
| Histogramming | hist, boost-histogram | ND axes for systematic variations. NOT ROOT TH1/numpy.histogram for filling. |
| Statistical model | pyhf, cabinetry | HistFactory JSON workspaces; cabinetry for ranking/pulls. NOT RooFit/RooStats/custom. |
| Unbinned fits | zfit | When binned HistFactory is insufficient. |
| MVA | xgboost, scikit-learn | BDTs via xgboost; sklearn for preprocessing/metrics. |
| Plotting | matplotlib, mplhep | See `04-output.md` for figure standards. NOT ROOT/plotly. |
| Columnar model | coffea | `NanoEvents`, `PackedSelection`. Optional. |
| Jet clustering | fastjet | e+e−: Durham (`ee_genkt_algorithm`, p=1). pp: anti-kt. NOT manual clustering. |
| b-tagging | tiered (below) | Agent builds taggers in Phase 2. |
| Logging | logging + rich | No bare `print()`. See §11. |
| Documents | pandoc (≥3.0) + pdflatex | Markdown → PDF. Never use LLM for conversion. |
| Dependencies | pixi | `pixi.toml` is single source of truth. NOT pip/conda. |
| Experiment knowledge | RAG (SciTreeRAG) | See §2.2. |

**Tiered tagging:** (1) cut-based cross-check — always exists; (2) BDT as
default primary; (3) NN only if the input space justifies it.

### 7.2 Paradigms
- **Prototype on a ~1000-event slice, then scale.** Full dataset is for
  production, not debugging.
- **Read the API before working around it;** add idioms to
  `06-appendix.md`.
- **Columnar analysis** — arrays + boolean masks, not event loops. Cuts are
  immutable named masks; never modify arrays.
- **Workspace as artifact** — pyhf JSON written to disk, committed.
- **Fit reproducibility** — each fit has a pixi task; human can re-run.
- **Plots are evidence** — every claim has a figure or table.
- **Pin random seeds; record software versions.**
- **MC normalization:** weight = σ·L / sum(w_generated) (algebraic sum for NLO).
- **Data-driven normalization:** when MC doesn't match data after proper
  scaling/calibration, add control regions with floating normalization
  parameters and let the fit absorb residual mismatches. Do not manually scale
  MC to match data.
- **Systematic naming:** `{source}Up` / `{source}Down` (pyhf/cabinetry).
- **Binning:** no bin with < ~5 expected events; variable binning when motivated.

### 7.3 Scale-Out
Estimate before running (input size, per-event cost on 1000-event slice, peak
memory). Prefer the simplest pattern that works; never wait >15 min on a login
node when SLURM exists. See `06-appendix.md` for SLURM templates.

| Estimated time | Mode |
|----------------|------|
| < 2 min | Single-core local |
| 2–15 min | `ProcessPoolExecutor` or multicore |
| > 15 min | SLURM: `sbatch --wait` (single) or `--array` (per-file) |

### 7.4 Multiprocessing Safety
With `ProcessPoolExecutor` and threading libraries (fastjet, ROOT, numpy+MKL),
always set the start method to `forkserver` or `spawn`:

```python
import multiprocessing
multiprocessing.set_start_method("forkserver", force=True)
```

The default `fork` copies parent thread state into children; active thread pools
(OpenMP, MKL) can return cached parent data — output looks plausible but is
numerically wrong. **Validation rule:** after parallel processing producing N
independent outputs, verify they are not bit-for-bit identical; if they are,
it's a multiprocessing bug, not a physics result.

### 7.5 Calibration Verification
**Every calibration must have a before/after comparison plot** (beam-spot to d0,
energy scale to jet pT, tracking SF, MC reweighting). The correction must
visibly improve the relevant distribution (narrower residuals, better data/MC,
reduced bias). If it *widens* a distribution or *worsens* data/MC, the
correction is wrong (e.g. the quantity was already corrected at reconstruction —
double-correction). Critical for archival data with undocumented processing
chains. Do not apply corrections without verifying their effect.

---
