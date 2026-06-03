## 11. Version Control and Coding Practices

### 11.1 Git

**Conventional commits:** `<type>(phase): <description>`. Types: feat, fix,
data, plot, doc, refactor, test, chore. **Commit after every meaningful
step** — commits are resume checkpoints if the agent crashes. **Branch per
phase** (`phase1_strategy`, etc.); merge to main after review passes.

### 11.2 Code Quality

- **KISS/YAGNI.** Scripts, not frameworks. No CLIs, config systems, plugins.
- **`__file__`-relative output paths.** Resolve outputs relative to the
  script file, not CWD, so output is identical regardless of shell CWD:
  ```python
  HERE = Path(__file__).resolve().parent
  OUT = HERE.parent / "outputs"
  FIG = OUT / "figures"
  ```
  CWD-relative paths break when invoked from a different directory.
- **Columnar.** Arrays + masks, not event loops.
- **Logging, not printing.** `logging` + `rich.logging.RichHandler`. Ruff
  `T201` enforces no `print()`. Standard setup:

```python
import logging
from rich.logging import RichHandler
logging.basicConfig(level=logging.INFO, format="%(message)s",
                    handlers=[RichHandler(rich_tracebacks=True)])
log = logging.getLogger(__name__)
```

- **Ruff + pre-commit** for formatting and linting on every commit.

### 11.3 Testing

Focus on **structural bugs** (wrong branch, wrong weight, inverted cut) —
catastrophic because they require re-running everything and produce
plausible-looking wrong numbers. **Always:** smoke test per phase (~100
events, no crashes); integration test (output files exist, correct
structure, no NaN). **Check:** variable names → right quantities; cut
inversions → correct complement; object efficiencies → consistent with
published; cutflow → monotonically decreasing; systematic variations →
expected direction. **Parallel outputs not identical:** if a parallel step
produces N independent results, verify they are not bit-for-bit identical —
identical outputs from independent inputs indicate a fork/threading bug.

### 11.4 Separation of Analysis and Plotting

**Analysis code and plotting code must be independent.** Analysis scripts
produce machine-readable artifacts (JSON, NPZ, CSV) — histograms, yields,
fit results, systematic shifts. Plotting scripts read those artifacts and
produce figures. The two must never be entangled in the same script.

Benefits: (1) plots are rerunnable — tweaking a legend/axis/color does not
re-run the analysis chain; `pixi run plot-X` regenerates from the saved
artifact in seconds. (2) Artifacts are the handoff — downstream phases and
the AN read artifacts, not figures. (3) Review is cleaner.

**In practice:**
- Analysis scripts write to `outputs/` (JSON, NPZ): yields, efficiencies,
  fit parameters, systematic shifts, histograms.
- Plotting scripts read from `outputs/`, write to `outputs/figures/`
  (PDF, PNG). Each has its own pixi task (e.g., `p3-plots`, `p4a-plots`);
  the `all` task runs analysis first, then plots.
- Plotting scripts must NOT call `uproot.open()` or process ROOT files
  directly. If a plot needs data not in the artifacts, extend the analysis
  script — do not bypass the artifact layer.

**Exception:** Quick data/MC overlay plots during Phase 2 exploration may
read ROOT files directly. From Phase 3 onward, separation is mandatory.

### 11.5 Task Graph

**Every script → pixi task.** `pixi run all` reproduces the full analysis
from raw data. Task names human-readable. Scripts idempotent (fixed seeds,
fixed output paths). The `all` task must: run every script in correct order
(not just the final step); be idempotent; include systematic variation
reruns, not just nominal; produce all figures, tables, and machine-readable
outputs; complete without manual intervention. Document its execution graph
in the AN reproduction contract appendix (see `04-output.md` → Reproduction
contract). Split scripts exceeding ~5 min into stages with intermediate
outputs. Update `pixi.toml` whenever scripts change.

### 11.6 Debug Code and Diagnostic Outputs

Debug scripts are prefixed `debug_` or placed in `scratch/`; never in the
`all` task or reproducibility chain. **But debug outputs are valuable —
preserve them.** Debug plots, diagnostic tables, intermediate sanity checks,
and exploratory figures must NOT be deleted or "cleaned up before review."
Save them to `logs/` or `outputs/debug/` and reference in the experiment
log. Anything that informed a decision must be traceable.

What goes where:
- **`outputs/`** — production artifacts (JSON, NPZ); enter the chain and AN.
- **`outputs/figures/`** — publication-quality figures for the AN.
- **`outputs/debug/`** — diagnostic figures and intermediate outputs; not
  in the AN, preserved and referenced in the experiment log (alternative
  binnings, failed fits, correlation checks, decision-driving plots).
- **`logs/`** — session logs, experiment-log entries; narrative record.

---

## 12. Scope Management and Downscoping

Downscoping is a last resort when the full-strength approach is genuinely
infeasible — not a shortcut for difficulty, not the default for a hard
problem. Every downscope weakens the analysis; the protocol ensures the
weakening is documented, quantified, and justified.

### When to downscope

Justified only when the stronger approach has been **attempted and failed**
or is **demonstrably infeasible** (not merely difficult/uncertain). Triggers:
data or MC genuinely unavailable; insufficient MC statistics after exploring
all samples; compute limits making the method impossible within the resource
envelope; missing external inputs with no viable substitute; method attempted
and shown to fail (documented). "The alternative might not work" / "would be
harder" are not sufficient — try the stronger method first; if it fails,
document why and downscope with evidence.

### How

0. **Attempt the full-strength approach first.** Either (a) attempt the
   stronger method and document its failure, or (b) document why attempting
   it is infeasible (not merely difficult). "We tried it and it failed
   because [reason]" is evidence; "we expected it wouldn't work" is not.
1. **Document** the constraint in the experiment log.
2. **Choose best achievable method.** Fall back along the complexity ladder
   (GNN → BDT → cut-based) or reduce scope. **Label the status change**:
   when a method's role changes, record the transition with a [D] label,
   e.g. `[D] SVD unfolding: co-primary → cross-check (diagonal fraction
   24.7%, below 30% threshold)`. The Phase 3/4 artifact must use the updated
   label. Silent status changes are Category A at review.
3. **Quantify impact.** Estimate what the missing resource would contribute.
4. **Carry to the AN.** Every downscope → method section + systematic table
   + Future Directions. A limitation only in the experiment log is not
   properly documented.

### Key scenarios

- **Missing MC:** omit if small, or estimate from theory (sigma × epsilon
  from a similar process).
- **Low MC stats:** coarser binning, merged regions, cut-and-count; include
  MC stat uncertainty (Barlow-Beeston).
- **Cannot evaluate a systematic from own data:** never leave as zero. Use a
  literature value (via RAG), inflate conservatively, cite the source.
- **Skipping approach exploration:** choosing a simpler approach (e.g.,
  cut-based over MVA) without trying the alternative is a downscope —
  document the constraint, quantify expected impact, carry to the AN.
  Concerns about the alternative's costs are valid constraints to document
  but do not exempt quantifying what was foregone.

### Review

Reviewers check: (1) was the stronger approach attempted, or is
infeasibility documented with evidence? (2) is the quantified impact
credible? (3) is the limitation documented in the AN? Downscoping without
evidence of attempting the stronger method is Category A.

### Future Directions — implement, don't defer

**The default response to a feasible improvement is to implement it, not to
write it in Future Directions.** When an agent identifies an improvement
(better tagging, generator comparison, calibration reducing a dominant
systematic), ask: "Can this be done in < 2 hours of implementation +
compute?" If yes, do it now. If no, document in Future Directions with a
specific explanation of what makes it infeasible.

**Future Directions IS for genuinely infeasible improvements:** collecting
more data (new running); developing a new algorithm architecture (R&D);
full detector simulation (multi-day compute + expertise); external inputs
not available (other groups); methods requiring software not installable in
the current environment (after documented installation failure).

**Future Directions is NOT for** tasks that take ~1–2 hours on existing
inputs — e.g. running PYTHIA 8 at particle level (~30 min), a contamination
matrix correction on an existing tagger (~1 hr), decomposing a systematic
into normalization vs shape (~1 hr), overlaying published measurements
(~1 hr), a per-hemisphere truth label from available gen-level info (~1 hr),
a data-driven calibration of an uncalibrated variable (~2 hr). These were
deferred in real analyses but later implemented in ~1 hour for significant
gains.

**Practical test:** "If the human said 'do it now,' could the agent complete
it within the current session?" If yes, it belongs in the current plan, not
Future Directions. The orchestrator monitors accumulating items and triggers
implementation when feasible. Phase 5 AN must include a Future Directions
section for genuinely infeasible items: what was downscoped, resources
needed, expected improvement, priority order. Each item must pass the
feasibility test; reviewers flag any item that could have been implemented.

---

## 4. Blinding / Staged Validation Protocol

Both searches and measurements follow the same staged protocol. For searches,
"blinding" means not examining the SR discriminant in data; for measurements
without SR structure, not computing the final quantity on real data. The gate
structure is identical.

### 4.1 Stages

| Stage | Data access | Gate |
|-------|-------------|------|
| Phases 1–3 | MC only (searches: no SR data; measurements: no data-derived result) | — |
| Phase 4a | Asimov/MC pseudo-data only | 1 reviewer (§6.2) |
| Phase 4b | 10% data subsample (fixed random seed) | 1 reviewer → human gate |
| Phase 4c | Full data | 1 reviewer |

**Asimov data** = synthetic pseudo-data from the nominal model, bin contents
at exact expected values (no fluctuations).

**10% partial unblinding (Phase 4b):** select 10% of data with a fixed seed,
normalize MC to 10% luminosity, run the full chain, compare to Phase 4a
expected results (should be compatible within large uncertainties). Fix
problems *before* seeing more data.

**Full unblinding (Phase 4c):** only after the human approves at the 4b gate.
Post-unblinding modifications must be documented and justified.

### 4.2 Human Gate

After Phase 4b review passes, present to the human: the draft analysis note
with 10% results, plus the unblinding checklist:

1. Background model validated (closure tests pass)
2. Systematics evaluated, fit model stable
3. Expected results physically sensible
4. Signal injection / closure tests pass
5. 10% partial unblinding shows no pathologies
6. All agent review cycles resolved (reviewer PASS)
7. Draft AN reviewed and publication-ready modulo full results

The human approves, requests changes, or halts. The agent does not fully
unblind autonomously.

---

## 9. Multi-Channel Analyses

Many HEP analyses involve multiple channels with different final states (e.g.,
ttH 0/1/2-lepton, or Zh vv+bb / ll+bb / qq+bb). Handled as follows:

**Phase 1 (Strategy)** defines the channel decomposition: which channels are
included (with physics motivation); how they are defined to ensure no event
overlap (orthogonal selections); which calibrations and systematics are shared
across channels (e.g. jet energy corrections, b-tagging, luminosity); which
are channel-specific (e.g. lepton ID for the leptonic channel only). The
channel structure is a strategy decision evaluated by the strategy review.

**Phases 2–3** may proceed per-channel, potentially in parallel: each channel
has its own exploration, selection, and background modeling; shared
calibrations are developed once and referenced by all; each channel produces
its own artifact (e.g. `SELECTION_CHANNEL_A.md`); a consolidation artifact
documents the overlap check (no shared events) and cross-channel consistency.

**Shared sub-analyses (calibrations):** components that are mini-analyses
producing calibration artifacts consumed by all channels — e.g. jet energy
corrections (correction factors + residual uncertainty), b-tag calibration
(scale factors + uncertainties per working point), trigger efficiency
(turn-on curves + uncertainties), luminosity (provided, may need validation).
Each shared sub-analysis: has its own experiment log and artifact (e.g.
`CALIBRATION_<NAME>.md`); follows phase structure (method, results,
validation, code); produces **central values + uncertainties** consumed as
inputs, with uncertainties propagating into Phase 4 as fit systematics;
**must demonstrate its effect** via before/after comparisons (improved mass
peak resolution/position, better data/MC agreement — the most informative
demonstration for what is calibrated).

Calibrations get no dedicated review gate. Their quality is validated by
(1) the artifact's own before/after plots (self-evident validation) and
(2) downstream phase reviews (Phase 3 selection, Phase 4a inference) flagging
problems via upstream feedback or regression — a suspicious scale factor or
uncertainty surfaces as data/MC disagreement or fit pathology downstream.
Shared sub-analyses are identified in Phase 1 (strategy) or Phase 2
(exploration, when needed calibrations are discovered); they run in parallel
with or before channel-specific Phase 2–3 work and may use dedicated sessions.

**Phase 4** combines channels in a single statistical model: the fit includes
all channels simultaneously; correlated systematics (including shared
calibrations) use shared nuisance parameters across channels; the combined
expected sensitivity is the primary figure of merit; per-channel results are
reported for diagnostics.

---
