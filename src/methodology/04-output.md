## Analysis Note Specification

The AN is versioned across Phases 4a–5: 4a writes the complete AN v1 with ALL detail (every diagnostic, systematic subsection, cross-check, comparison) using expected/Asimov results; 4b→10% numbers; 4c→full data; Phase 5 polishes + typesets. Phase-stamped, never overwritten; the executor both writes and typesets.

**Gold standard / completeness test:** a physicist who never saw the analysis reproduces every number from the AN alone. If a reader needs the code to understand a choice or a systematic, the AN has a gap.

**Notation consistency:** one consistent symbol per quantity throughout (same quantity under different names is Category A). **No empty sections:** every `##`/`###` heading has ≥2–3 sentences of prose before any figure/table (Category A). **Length:** the complete record — typically ~50–100 pages; under 30 means detail is missing (Category A) unless the analysis is genuinely simple AND the completeness test passes. (Honor the physics prompt: a quick/short analysis may target a concise note — that overrides the page-count rule when the prompt asks for it.) Rule of thumb: every cut → a before/after distribution; every systematic → an impact figure; every cross-check → a comparison plot.

**Change Log** (`# Change Log {-}`, unnumbered, first content after the TOC): reverse-chronological, grouped by phase/version, ≤1 rendered page (condense old phases to one-liners). A navigation aid, not a process diary.

### Required sections
1. **Introduction** — motivation, observable definition, prior measurements (cite ≥2 same/related measurements from other experiments where they exist).
2. **Data samples** — experiment, √s, **luminosity (mandatory; estimate L = N_had/σ_had if unpublished)**, MC generators, event counts. Use structured tables: a data table (one row per era, with L) and an MC table (one row per process: generator, σ, N_gen, k-factor).
3. **Event selection** — every cut with motivation, distribution plot (N-1), per-cut + cumulative efficiency.
4. **Corrections/unfolding** (measurements) — full procedure with **displayed key equations** (`$$...$$` for the correction formula, the likelihood/chi2, the systematic-propagation formula — implementable without the code); closure/stress tests documented with what/expected/observed(chi²/ndf,p)/figure/interpretation; stress-test reweighting formula + magnitudes + recovery.
5. **Systematic uncertainties** — one subsection per source in running prose: physical origin; evaluation method (what is varied, propagation chain, formula/reference, cited variation size — "±50%" is Category A unless measured); numerical impact (table row + bin-by-bin impact figure; flat shifts on a shape measurement are Category A); interpretation. Plus an **error-budget narrative** (which dominate, stat- vs syst-limited, resolving power).
6. **Cross-checks** — each within the section it validates, as comparison plots (overlay/ratio/pull, not pass/fail) with chi²/p-value + interpretation.
7. **Statistical method** — likelihood, fit validation, GoF.
8. **Results** — full uncertainties, per-bin tables, summary figures.
9. **Comparison to prior results and theory** — quantitative (chi²/ndf or pull with full covariance; overlay published points on the result figure). "Consistent with published" without a number is Category B.
10. **Conclusions.** 11. **Future directions** (§12). 12. **Known limitations** (3–5 most significant: what, attempted?, quantitative impact, fix). 13. **Appendices** — per-bin systematic tables, covariance matrices, extended cutflow, **limitation index** ([A]/[L]/[D] registry), **reproduction contract** (exact `pixi` command sequence + DAG to reproduce every number).

### Statistical methodology standards (Category A if violated)
- **Full covariance mandatory** when a covariance matrix exists: $\chi^2 = (d-m)^T C^{-1} (d-m)$. Diagonal-only may be shown for reference but never as primary.
- **Pull diagnostics:** state expected |pull|>2σ (4.6%) and >3σ (0.27%); quote pull RMS (1.0±0.1 healthy; <0.7 overcoverage; >1.3 undercoverage) — both extremes need discussion.
- **GoF for the primary result:** chi²/ndf < 3 (p > 0.01); p < 0.01 is Category A unless the source is identified, shown not to bias the result, and an acceptable-GoF config is shown as a cross-check. Picking best-precision while ignoring GoF is forbidden.
- **Closure passes at p > 0.05** (ad-hoc "chi²/ndf < 5" thresholds are invalid); p < 0.01 = failed, needs 3+ remediation attempts.

### Standard diagnostic figures (missing = Category A when applicable)
Per-variable data/MC (every selection + MVA variable); per-category validation grids; per-cut distributions (N-1 + cut-variation sensitivity); MVA diagnostics (ROC+AUC, train/test scores, feature importance, data/MC on the score, alternative architecture); per-systematic impact figures; cross-check figures (overlay/ratio/pull); fit diagnostics (NP pulls pre/post-fit, GoF, post-fit data/model); a **mandatory comparison overlay** (measurement + published values on the SAME axes with a ratio/pull panel and chi²/ndf — no overlay is Category B, "consistent" without showing it is Category A); a **systematic breakdown figure** (waterfall/bar — a table alone is insufficient). Methodology diagrams where the figure-scrolling test exposes a gap.

### Requirements
Self-contained, machine-readable `results/` (CSV/JSON for spectra + covariance). **BibTeX:** `[@key]` with `references.bib`, entries with `doi`/`url`/`eprint` — **never generate BibTeX from training data** (fabricated keys/authors/journals); every entry traceable to a real source (`get_paper`/DOI/INSPIRE/the PDF) or it is Category A. **Reference count:** < 15 in a 50+ page AN is Category A, < 20 is Category B. Cite foundational theory, reference analyses, detector papers, methodology, and PDG/world averages. Phase-1 extracted published numerical results are binding §6.8 targets at 4c (don't defer lookup to Phase 5).

### Consistency across phases
When a primary config/operating point changes between 4a and 4c: state it explicitly in the results section (not only the Change Log); re-evaluate all systematics at the new point (or justify transfer quantitatively); make labels consistent everywhere (search the body for the old label); never compute pulls across different operating points without flagging. Single post-selection event count used consistently (unexplained mismatches are Category B). Consistent rounding; verify quadrature totals match displayed components.

---

## LaTeX compilation

Markdown → PDF in three steps: **pandoc** (≥3.0) → `.tex`; **`postprocess_tex.py`** (deterministic fixes: title math, escaped standalone math, margins, abstract→environment, references unnumbering, table spacing, short longtable→table, FloatBarrier, needspace, duplicate header/label removal, appendix, clearpage); **tectonic** (or xelatex) → PDF. The `build-pdf` pixi task runs this. The preamble (`conventions/preamble.tex`) sets default image height `0.45\linewidth`, narrowed captions, relaxed float placement, widow/orphan penalties — do not modify per-analysis without reason. Never use an LLM for LaTeX conversion.

### Pandoc pitfalls (mandatory)
- **Never use `$\pm$`, `$<$`, `$>$`, `$-$`, `$\sim$` as standalone math** — use Unicode `± < > − ~` (bare single-symbol math renders with visible dollar signs and breaks pandoc-crossref). `$...$` only for real expressions (`$M_Z$`, `$\chi^2/ndf$`).
- **YAML title doesn't render math** — use Unicode (`√s = 91.2 GeV`).
- **No `\mathrm{}` in captions/headers** (pandoc alt-text error); no `@ref` inside `$...$`; no complex LaTeX in section headers.
- Abstract must be unnumbered (auto-fixed); references unnumbered (auto-fixed); tables need spacing before them (auto-fixed).

### Tables and rendering
Keep columns narrow (abbreviations; long text → prose/footnotes); avoid monospace (paths overflow); split wide tables (>6 cols); consistent numeric precision (2–3 sig figs). After compiling, `grep "Overfull.*hbox" *.log` — any overfull hbox on a figure/table is Category A. Check: no orphaned headings (`\needspace`), figures not clipped, short tables not split, all `@fig:`/`@tbl:`/`@eq:` resolve, abstract before TOC.

---

## Plotting (Category A unless noted) — these rules are gospel; lint enforces them

Base: `mh.style.use("CMS")`; **`figsize=(10,10)` LOCKED** (ratio plots `(10,10)` with `height_ratios=[3,1]`; grids = 10in per row/col; any custom figsize is Category A). Save **both** PDF and PNG with `bbox_inches="tight", dpi=200, transparent=True`; `plt.close(fig)` after. Never `tight_layout()`/`constrained_layout`. `np.random.seed(42)` if any randomness.

- **Experiment label on every independent axes** via `mh.label.exp_label(exp="<EXP>", data=True, llabel="Open Data"|"Open Simulation", rlabel=r"$\sqrt{s}=X$", loc=0)` — **main panel only on ratio plots, NEVER the ratio panel.** Always `data=True` with explicit `llabel`; labeling as just "<EXP>" without "Open Data"/"Open Simulation" implies an official result (Category A); never `data=False` with `llabel` (produces "Simulation Open Simulation").
- **No `ax.set_title()`** (captions go in the AN). **No numeric `fontsize=`** (CMS stylesheet sets sizes; relative `'x-small'` etc. allowed; legends `fontsize="x-small"`). **No raw `ax.text()`/`ax.annotate()`** on data plots (use `mh.label.add_text`); exception: conceptual diagrams.
- **Legend must not overlap data** — `from mplhep.plot import mpl_magic; mpl_magic(ax)` after plotting; manual placement only when a genuinely empty region exists. Overlap is Category A.
- **Human-readable labels** — no code identifiers / bare `_` outside `$...$` math (Category A).
- **Histograms:** `mh.histplot()` for binned data — never `ax.step()`/`ax.bar()`/`ax.fill_between()`. Raw counts → `histplot` gives sqrt(N) errors. **Derived quantities (normalized dists, ratios, efficiencies, correction factors, systematic shifts) MUST pass explicit `yerr=`** — without it mplhep applies sqrt(bin-content) (e.g. 570% error bars). Test: filled via `h.fill(values)` → auto-errors OK; assigned via `h.view()[:]=...` or computed → MUST pass `yerr=`. Weighted fills with `Weight()` storage have correct auto-errors.
- **2D colorbars:** `mh.hist2dplot(H, cbarextend=True)` or `cax = mh.utils.make_square_add_cbar(ax); fig.colorbar(im, cax=cax)`. **`fig.colorbar(im)`, `fig.colorbar(im, ax=ax)`, `plt.colorbar(...)` are Category A** (they steal main-axes space). Set `ax.set_aspect('equal')` for 2D where both axes are the same coordinate type.
- **Ratio plots:** `sharex=True` AND `fig.subplots_adjust(hspace=0)` are both non-negotiable (missing either is Category A); hide main x-tick labels and set ratio y-limits so boundary ticks don't collide; remove the spurious "Axis 0" text.
- Round bin-width labels to clean values; suppress matplotlib "1e6" offset notation; log y-scale when spanning >2 orders of magnitude; tight axis limits.
- **Sanity:** if two supposedly-independent distributions look identical, investigate (bug / tautology / genuine correlation) — don't silently accept; a single source >100% relative in a systematic-breakdown bin → investigate (usually a bug).

**Pre-commit self-verification (executor's job, reviewer is the safety net).** Grep for `ax.step(`, `ax.bar(`, `ax.text(`, `tight_layout`, `plt.colorbar`/`fig.colorbar(...,ax=`, `set_title(`, numeric `fontsize=`, and the `.view()[` + `histtype="errorbar"` without `yerr=` trap. Then visually inspect every rendered PNG: legend not overlapping, text readable, exp_label on main panel only, no leaked identifiers, histplot style, tight ranges, colorbars same height as axes, real data present, derived-quantity error bars plausible (not spanning the y-axis). `pixi run lint-plots` mechanizes the grep.

**Error propagation (pass `yerr=` explicitly):** normalized `(1/N)dN/dx`: `sqrt(n)/(N·dx)`; ratio `A/B`: `R·sqrt((σ_A/A)²+(σ_B/B)²)`; efficiency `k/n`: Clopper-Pearson; bin-width-normalized: `sqrt(n)/dx`.

**Captions** (self-contained, 2–4 sentences, `<What> (<where>) <context + conclusion>`): name the observable, identify panels for composites ("(left)"), state context not visible (selection/normalization/what it validates), give the key conclusion. Do not restate axes/legend. Under two sentences is Category A.

**Figure compositing (in LaTeX only — never PIL/matplotlib stitching).** Group related figures side-by-side with `\includegraphics[height=...]` (use `height=`, not `width=`, so aspect ratios align) + `\hspace`, ONE unified caption, no `\subfloat`, no (a)/(b) sub-captions (use "(left)"/"(right)" in the caption). Per-panel heights: 1×2 `0.38\linewidth`, 2×2 `0.32`, 1×3/2×3 `0.30`, 3×3 `0.28`. Merging per-variable/per-systematic/per-cut runs and nominal+uncertainty pairs is mandatory (Category A if left as standalone runs). After compositing, inspect the PDF for equal panel sizes and a clean grid.

**Cross-references:** `![Caption](figures/name.pdf){#fig:name}` + `@fig:name` (sentence start: `Figure @fig:name`); never `[-@fig:...]`; tables `{#tbl:}`/`@tbl:`, equations `{#eq:}`/`@eq:`. **Correlation/covariance:** matrix heatmaps (`hist2dplot`/`pcolormesh`, diverging `RdBu_r` centered at 0, `vmin=-1,vmax=1` for correlations), never overlaid 1D.

**Conceptual diagrams** (analysis flow, correction chains, region definitions): embedded in the relevant section, `mh.style.use("CMS")`, no `exp_label`, `figsize` free, built with matplotlib patches/arrows/`ax.text` (the one exception to the no-`ax.text` rule), PDF+PNG. Identified in Phase 1, produced in Phase 5; encouraged, judged by the figure-scrolling test.

**Plotting delegation:** a plotting subagent receives this section + the data (paths/arrays) + plot kind + axis labels/ranges + exp_label params + output path; it applies the template and makes no physics decisions.
