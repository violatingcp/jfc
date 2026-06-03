# Phase 2: Exploration

Explore the data and MC for a **{{analysis_type}}** analysis.
**Start in plan mode** (which files, which variables, which plots), then execute. Keep it short — prototype on ~1000-event slices, not the full data.

## Output artifact
`outputs/EXPLORATION.md` — sample inventory/schema, data-quality assessment, variable survey + ranking, baseline yields.

## Key requirements
- **Schema inventory:** for each file — tree/branch names+types, #events, cross-section, luminosity. This is the contract later phases build on.
- **Data archaeology (archived/open data):** check weight/flag branches for non-trivial values; compare counts to σ×L to detect pre-selection; note generator/tune/energy coverage and any truth-level info. If a discovery changes feasibility, flag it as a **strategy-revision input** ("Phase 1 assumed X; Phase 2 found Y; implication Z").
- **Data quality:** NaN/Inf, unphysical values, empty branches — document.
- **Variable survey:** signal-vs-background distributions for candidate discriminants, ranked by separation (ROC AUC or S/√B).
- **Baseline yields:** event counts after preselection, normalized, for data and each MC sample.
- **PDF toolchain smoke test:** build a tiny stub note with `pixi run build-pdf`, confirm it compiles, delete the stub.

## Plotting (lint before finishing: `pixi run lint-plots`)
mplhep CMS style; `figsize=(10,10)`; `mh.histplot()`; `exp_label` on main axes only; ratio panels `hspace=0`; no titles; no absolute font sizes; save PDF+PNG; `logging`+`rich`, no bare `print`.

## Review
**Self-review** (no separate reviewer). Confirm: schema complete, data quality checked, variable survey + yields present, PDF toolchain works, figures pass lint, experiment log updated. Write to `review/self/`.
