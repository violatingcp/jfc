## 11. Version Control and Coding Practices

### 11.1 Git
Conventional commits `<type>(phase): <description>` (feat/fix/data/plot/doc/refactor/test/chore). Commit after every meaningful step — commits are crash-resume checkpoints. Branch per phase; merge to main after review.

### 11.2 Code quality
- **KISS/YAGNI** — scripts, not frameworks (no CLIs/config systems/plugins).
- **`__file__`-relative output paths** (`HERE = Path(__file__).resolve().parent`), so output is identical regardless of CWD.
- **Columnar** — arrays + masks, no event loops.
- **Logging, not printing** — `logging` + `rich.logging.RichHandler` (Ruff T201 forbids `print`):
  ```python
  import logging
  from rich.logging import RichHandler
  logging.basicConfig(level=logging.INFO, format="%(message)s",
                      handlers=[RichHandler(rich_tracebacks=True)])
  log = logging.getLogger(__name__)
  ```
- Ruff + pre-commit on every commit.

### 11.3 Testing
Focus on **structural bugs** (wrong branch, wrong weight, inverted cut) — they re-run everything and produce plausible wrong numbers. Always: per-phase smoke test (~100 events, no crash) + integration test (files exist, right structure, no NaN). Check variable→quantity mapping, cut complements, efficiencies vs published, monotonic cutflow, systematic directions. **Parallel outputs that are bit-identical across independent inputs indicate a fork/threading bug.**

### 11.4 Separation of analysis and plotting
**Analysis and plotting code must be independent.** Analysis scripts write machine-readable artifacts (JSON/NPZ/CSV) to `outputs/`; plotting scripts read those and write figures to `outputs/figures/` — never entangled. Plotting scripts must NOT call `uproot.open()`; if a plot needs data not in the artifacts, extend the analysis script. (Exception: quick Phase-2 data/MC overlays may read ROOT directly; from Phase 3 on, separation is mandatory.) Each gets its own pixi task; `all` runs analysis then plots.

### 11.5 Task graph
**Every script → a pixi task; `pixi run all` reproduces the full analysis from raw data** in correct order (not just the final step), idempotent (fixed seeds + output paths), including systematic-variation reruns, producing all figures/tables/machine-readable outputs without manual intervention. Split scripts > ~5 min into staged steps. Update `pixi.toml` whenever scripts change.

### 11.6 Debug outputs
Debug scripts are `debug_`-prefixed or in `scratch/`; never in the `all` chain. But preserve diagnostic outputs — anything that informed a decision must be traceable. `outputs/` = production artifacts; `outputs/figures/` = AN figures; `outputs/debug/` = diagnostics (referenced in the experiment log); `logs/` = session/experiment-log narrative.

---

## 12. Scope Management and Downscoping

Downscoping is a last resort when the full-strength approach is genuinely infeasible — not a shortcut for difficulty. Every downscope weakens the analysis; document, quantify, and justify it.

**When:** only when the stronger approach was **attempted and failed** or is **demonstrably infeasible** (data/MC unavailable; insufficient MC stats after exploring all samples; compute genuinely beyond the envelope; missing external inputs with no substitute). "Might not work" / "would be harder" is not sufficient — try first.

**How:** (0) attempt the full-strength method (document the failure) or document why attempting is infeasible; (1) record the constraint in the experiment log; (2) fall back along the complexity ladder (GNN→BDT→cut-based) and **label the status change with a [D]** (e.g. `[D] SVD unfolding: co-primary → cross-check (diagonal 24.7% < 30%)`) — silent status changes are Category A; (3) quantify what the missing resource would contribute; (4) carry to the AN (method section + systematic table + Future Directions). A limitation only in the experiment log is not properly documented.

**Scenarios:** missing MC → omit if small or estimate from theory (σ·ε of a similar process); low MC stats → coarser binning / cut-and-count + Barlow-Beeston; a systematic not evaluable from own data → never zero, use a cited literature value inflated conservatively; skipping approach exploration (e.g. cut-based over MVA) is itself a downscope — document and quantify the foregone gain.

**Review:** was the stronger approach attempted (or infeasibility evidenced)? is the quantified impact credible? is the limitation in the AN? Downscoping without evidence of attempting the stronger method is Category A.

**Future Directions — implement, don't defer.** Default response to a feasible improvement is to implement it now. Test: "If the human said 'do it now,' could the agent finish within the session?" If yes (≲1–2 hr on existing inputs — e.g. particle-level PYTHIA, a contamination-matrix correction, decomposing a systematic into norm vs shape, overlaying published measurements), do it. Future Directions is only for genuinely infeasible items (new data-taking, new algorithm R&D, multi-day full simulation, external inputs from other groups, software not installable after a documented failure) — with resources needed, expected gain, and priority. Reviewers flag any deferred item that could have been implemented.

---

## 4. Blinding / Staged Validation Protocol

Searches and measurements follow the same staged protocol — "blinding" = not examining the SR discriminant in data (searches) or not computing the final quantity on real data (measurements).

| Stage | Data access | Gate |
|---|---|---|
| Phases 1–3 | MC only | — |
| Phase 4a | Asimov/MC pseudo-data only | reviewer |
| Phase 4b | 10% data subsample (fixed seed) | human gate (no reviewer) |
| Phase 4c | full data | orchestrator checklist (no reviewer) |

**Asimov data** = synthetic pseudo-data from the nominal model with bin contents at exact expected values (no fluctuations). **10% partial unblinding (4b):** select 10% of data with a fixed seed, normalize MC to 10% luminosity, run the full chain, compare to 4a expected (compatible within large uncertainties); fix problems before seeing more. **Full unblinding (4c):** only after the human approves at 4b; post-unblinding changes must be documented and justified.

**Human gate (after 4b):** present the draft AN with 10% results + the unblinding checklist — (1) background model validated, (2) systematics evaluated + fit stable, (3) expected results sensible, (4) signal-injection/closure pass, (5) 10% shows no pathologies, (6) all review cycles resolved, (7) draft AN publication-ready modulo full results. The agent does not fully unblind autonomously.

---

## 9. Multi-Channel Analyses

**Phase 1** defines the channel decomposition: which channels (with motivation), orthogonal selections (no event overlap), and which calibrations/systematics are shared (jet energy, b-tagging, luminosity) vs channel-specific (e.g. leptonic-only lepton ID).

**Phases 2–3** may run per-channel in parallel: each channel has its own exploration/selection/background artifact (e.g. `SELECTION_CHANNEL_A.md`); shared calibrations are developed once; a consolidation artifact documents the overlap check and cross-channel consistency.

**Shared sub-analyses (calibrations)** are mini-analyses producing central values + uncertainties consumed by all channels (jet energy, b-tag SFs, trigger turn-ons, luminosity). Each has its own log + artifact (`CALIBRATION_<NAME>.md`), follows the phase structure, and **must demonstrate its effect** via before/after comparisons. They get no dedicated review gate — quality is validated by their own before/after plots and by downstream reviews (a bad scale factor surfaces as data/MC disagreement or fit pathology).

**Phase 4** combines channels in a single statistical model: all channels fit simultaneously; correlated systematics (incl. shared calibrations) use shared nuisance parameters; the combined expected sensitivity is the primary figure of merit; per-channel results are reported for diagnostics.
