## Analysis Note Specification

The AN is a versioned document across Phases 4a–5. Phase 4a writes the
complete AN v1 with ALL detail — every diagnostic plot, every systematic
subsection, every cross-check, every comparison — using expected (Asimov/MC)
results only. Phase 4b updates numbers to 10% data; Phase 4c updates to full
data; Phase 5 polishes prose and typesets. Each phase produces a phase-stamped
version (`ANALYSIS_NOTE_{phase}_v{N}.md`); no version is ever overwritten.
Review/fix cycles increment the version within a phase. All versions are
preserved on disk. The executor both writes and typesets the AN.

**The gold standard: a physicist who has never seen the analysis should be
able to reproduce every number from the AN alone.** This is the completeness
test. If a reader needs the code to understand a choice, reconstruct the event
selection, or learn how a systematic was evaluated, the AN has a gap.

**Notation consistency.** Every physical quantity uses a single consistent
symbol throughout. The same variable under different names in different
sections (e.g., $f_s$ in formalism but $f_t$ in results) is Category A.
Define symbols at first use; note explicitly when adopting reference notation
that differs from earlier usage.

**Length.** The AN is the complete record — ~50-100 rendered pages for a
typical measurement. **Under 30 pages means detail is missing** — Category A
at Phase 5 unless the reviewer confirms the analysis is genuinely simple
(single observable, fewer than 5 systematic sources, no MVA, no unfolding)
AND the completeness test still passes. A fully documented 35-page AN is
acceptable; a complex analysis crammed into 40 pages is not.

Common causes of thin ANs: missing per-cut distributions (before/after each
cut); missing per-systematic impact figures; missing cross-check comparison
plots (just "PASS" is insufficient); missing MVA diagnostics when a classifier
is used (ROC, score distributions, feature importance, produced in Phase 3);
summary tables without supporting figures; methods in one sentence instead of
full paragraphs with equations.

**No empty sections.** Every section heading must be followed by at least 2-3
sentences of prose before any figure or table. A heading containing only a
figure with no introductory text is **Category A**. The executor must verify
every `##` and `###` heading has prose before any figure.

Rule of thumb: every selection cut needs a before/after distribution plot,
every systematic needs an impact figure, every cross-check needs a comparison
plot. A 5-energy-point lineshape fit should have ~30+ figures minimum.

---

### Change log

The AN includes a **Change Log** as the first content after the table of
contents, before the Introduction. It is unnumbered (`# Change Log {-}`) and
not in section numbering. It is maintained incrementally — each phase appends
entries at the top (reverse chronological), grouped by phase/version with
bulleted summaries of what changed and why. The first version (4a) initializes
with "Initial AN version (Phase 4a)."

**Change log length.** Must not exceed 1 rendered page. For multi-iteration
analyses, condense earlier phases to one-line summaries, keep full detail for
the last 2 versions, and move full history to an appendix if needed. It is a
navigation aid, not a process diary — internal phase labels, Finding numbers,
and agent debug details do not belong here.

Example format:
```
# Change Log {-}

**Phase 5 v2 (review fixes)**
- Expanded systematic §5.3 (tracking efficiency) with per-bin impact figure
- Fixed cross-reference to response matrix in §4.2

**Phase 4c v1**
- Updated all results to full dataset (was 10% in 4b)
- Added observed/expected comparison figures

**Phase 4a v1**
- Initial AN version (Phase 4a)
```

---

### Required sections

1. **Introduction** — motivation, observable definition, prior measurements.
2. **Data samples** — experiment, sqrt(s), luminosity, MC generators, event
   counts. Must include **structured sample tables** (not free-form prose or
   file listings):

   **Data summary table** — one row per data-taking era. **Integrated
   luminosity is mandatory** (one column per period). If not published for
   archived/open data, estimate from the hadronic cross-section and event
   count: $\mathcal{L} = N_\text{had} / \sigma_\text{had}$, and state the
   method. An AN without luminosity figures is **Category B**.

   | Period | $\sqrt{s}$ [GeV] | Events (pre-sel) | $\mathcal{L}$ [pb$^{-1}$] |
   |--------|-------------------|------------------|---------------------------|
   | 1992   | 91.2              | 450k             | 10.0                      |

   **MC sample table** — one row per physics process:

   | Process | Generator | $\sigma$ [nb] | $N_\text{gen}$ | $k$-factor | Notes |
   |---------|-----------|---------------|----------------|------------|-------|
   | $q\bar{q}$ | PYTHIA 6.1 | 30.5 | 4.0M | 1.0 | hadronic Z |

   These are summary-level (one row per era/process), not per-file
   inventories. Mark unknowns explicitly for open data.
3. **Event selection** — every cut with motivation, distribution plot (N-1
   preferred), efficiency (per-cut and cumulative), sensitivity to cut
   variation.
4. **Corrections / unfolding** (measurements) — full procedure, closure/stress
   tests, response matrix, regularization. **Key equations must be displayed**
   as `$$...$$`: the correction/unfolding formula, the likelihood or chi2, and
   the systematic propagation formula. A reader must be able to implement the
   method from the equations without the code. "Applied bin-by-bin" without
   showing $C(\chi) = N_\text{gen}(\chi) / N_\text{reco}(\chi)$ is incomplete.

   **Validation documentation standard.** Each validation test (closure,
   stress, flat-prior, alternative method) must include ALL of: (a) what was
   tested and why, (b) expected result, (c) observed (chi2/ndf, p-value, max
   deviation), (d) a figure (not just a number), (e) interpretation — does it
   pass? If not, what was tried (3+ attempts), and what it means for
   reliability. "Closure test passes" without (a–e) is Category B.

   **Stress test formula.** State the reweighting formula (e.g.,
   $w = 1 + \alpha(T - \langle T \rangle)/\sigma_T$), the variable, the
   magnitudes tested (5%, 10%, 20%), and recovery accuracy at each level. A
   magnitude-vs-max-bin-deviation table is the minimum.

   **Statistical methodology standards (Category A if violated).**
   - **Full covariance mandatory.** When a covariance matrix exists, ALL
     chi-squared tests MUST use it: $\chi^2 = (\mathbf{d} - \mathbf{m})^T
     C^{-1} (\mathbf{d} - \mathbf{m})$. Diagonal-only may be reported for
     reference but MUST NOT be primary (it underestimates chi2 for positively
     correlated bins, giving artificially good p-values). If ill-conditioned
     (condition number > $10^8$), note it and report both with caveats.
   - **Pull distribution diagnostics.** State the expected fraction with
     |pull| > 2σ (4.6%) and > 3σ (0.27%). Actual >> expected: uncertainties
     underestimated or genuine data-MC difference (state which). Actual <<
     expected or pull RMS < 0.7: overestimated (see overcoverage protocol §3).
     Quote pull RMS: 1.0 ± 0.1 healthy; < 0.7 overcoverage; > 1.3
     undercoverage. Both extremes require discussion.
   - **Goodness-of-fit for the primary result.** Primary extraction must have
     $\chi^2/\text{ndf} < 3$ ($p > 0.01$). $p < 0.01$ is Category A unless
     (a) the source is identified, (b) shown not to bias the extracted
     parameter, and (c) a configuration with acceptable GoF is shown as a
     cross-check. Picking best-precision while ignoring fit quality is
     forbidden.
   - **Closure test criteria.** Closure passes when $p > 0.05$. Ad hoc
     thresholds ("$\chi^2/\text{ndf} < 5$") are invalid (ndf-dependent). When
     $p < 0.01$, closure failed; 3+ remediation attempts required before
     accepting as a limitation.
5. **Systematic uncertainties** — one subsection per source: **Physical
   origin** (1-2 sentences); **Evaluation method** (what is varied, the
   propagation chain, the formula/reference; state and justify variation size
   with a measurement or published uncertainty — "±50% on the background" is
   Category A unless 50% IS the measured uncertainty); **Numerical impact**
   (table row + impact figure showing bin-by-bin shift; flat shifts on shape
   measurements are Category A); **Interpretation** (dominant? correlated?
   what reduces it?). A subsection stating only a number without the
   propagation chain is incomplete. Document failed evaluation attempts.

   **Error budget narrative (required).** After per-source subsections and the
   summary budget table, a narrative paragraph discussing: (a) which sources
   dominate and why, (b) statistically or systematically limited, (c) concrete
   improvements to reduce dominant sources, (d) **resolving power** — can it
   distinguish SM from a ±20% deviation at 2σ? If chi2/ndf << 1 (all pulls <
   0.5σ), explicitly discuss whether systematics are conservatively
   overestimated and the cost to discriminating power.
6. **Cross-checks** — each as a subsection within the section it validates
   (not standalone). Comparison plots (overlay, ratio, or pull — not pass/fail),
   chi2/p-value, interpretation. Examples: run-period stability, subdetector
   comparisons, alternative selections/correction methods, kinematic
   subsamples, generator comparisons. Large cross-checks → appendix with
   forward reference.
7. **Statistical method** — likelihood, fit validation, GoF.
8. **Results** — full uncertainties, per-bin tables, summary figures.
9. **Comparison to prior results and theory** — quantitative (chi2 with full
   covariance). Every comparison must state a number: chi2/ndf, pull in sigma,
   or ratio with uncertainty. "Consistent with published values" without a
   chi2 or pull is Category B. When prior measurements exist at the same
   kinematic points, overlay them on the result figure or give a bin-by-bin
   comparison table. Address: (a) comparison to the best published
   measurement, (b) is precision competitive/comparable/exploratory, (c) what
   it says about method validity.
10. **Conclusions** — result, precision, dominant limitations.
11. **Future directions** — concrete roadmap (§12).
12. **Known limitations and open questions** — concise, honest. For each of
    the 3-5 most significant limitations: what it is (one sentence); whether it
    was attempted (what was tried, what failed); quantitative impact ("shifts
    central value by < X%" / "inflates total uncertainty by Y%"); concrete fix.
    Distinct from the Limitation Index (a complete labeled registry [A1]/[L1]/
    [D1] in the appendix). Known Limitations is the physicist-facing narrative
    answering "what should I keep in mind when using these numbers?" Hiding all
    limitations in an appendix with no honest main-body assessment is
    Category B.
13. **Appendices** — per-bin systematic tables, covariance matrices, extended
    cutflow, auxiliary plots, **limitation index**, **reproduction contract**.
    The bulk of detail lives here.

The **limitation index** collects all constraints [A1], limitations [L1], and
decisions [D1] from Phase 1 onward. Each entry: label, one-line description,
where introduced, impact on result, mitigation.

The **reproduction contract** (required appendix) documents the exact command
sequence to reproduce the analysis from raw data to final result: environment
setup (`pixi install`, data paths); the exact pixi task sequence in execution
order with expected outputs per step; a workflow DAG diagram (inputs →
processing → outputs, with systematic variation branches); any manual steps;
expected runtime per step. It must be sufficient for an unfamiliar physicist to
reproduce every number by following the commands verbatim.

**Covariance matrix presentation** (appendix): per-source correlation matrices
(one panel per component — statistical, regularization, experimental,
theory/model, analysis-specific — all on the same color scale, each
interpreted physically); total covariance and correlation matrices (state max
off-diagonal $|\rho_{ij}|$ and which component dominates); a recommended fit
window for downstream extractions if some bins have unreliable covariance.

### Number and configuration consistency across phases

When a primary operating point, configuration, or method changes between
Phase 4a (expected) and Phase 4c (observed), the AN must:
1. **State the change explicitly** in the relevant results section (not only
   the Change Log).
2. **Re-evaluate ALL systematics at the new operating point**, or document
   quantitatively why the original evaluation transfers (e.g., "tracking
   systematic varies < 5% across κ, so κ = 0.5 evaluation is used").
3. **Ensure label consistency**: if §10 says "primary κ = 0.5" but §13 says
   "primary κ = 0.3", update or annotate. The Phase 5 executor must search the
   body for the old label and update/annotate every occurrence.
4. **Never compute pulls between results at different operating points**
   without flagging the comparison as approximate.

**Event count consistency.** A single dataset has a single post-selection
event count used consistently in the cutflow, body text, and summary tables.
Discrepancies (e.g., 2,889,000 vs 2,889,543) must be explained. Unexplained
mismatches are **Category B**.

**Rounding consistency.** Round uncertainties consistently everywhere (don't
mix "±0.01" and "±0.011"). When showing component uncertainties and a total,
verify $\sqrt{\sigma_1^2 + \sigma_2^2}$ matches the displayed total to the
displayed precision.

### Standard diagnostic figures

Required when applicable; most are produced in Phases 2–4 and aggregated in
Phase 5. Missing diagnostics are Category A.
- **Per-variable data/MC comparisons** — every selection variable and MVA
  training feature. Group into appendix grids when numerous.
- **Per-category validation grids** — when the observable combines multiple
  reconstructed object types, produce a complete kinematic validation grid
  (angular acceptance, momentum, quality variables, data/MC ratio) FOR EACH
  CATEGORY. Missing per-category validation for a category entering the
  observable is Category B.
- **Per-cut distributions** — N-1 preferred; include sensitivity to cut
  variation (e.g., ±10% shift).
- **MVA diagnostics** (when a classifier is used) — ROC with AUC, train/test
  score distributions (overtraining), feature importance, data/MC on
  classifier output, alternative architecture comparison.
- **Per-systematic impact figures** — how each source shifts the result.
- **Cross-check result figures** — overlay, ratio, or pull (not pass/fail).
- **Fit diagnostics** — nuisance pulls (pre/post-fit), GoF (chi2/ndf,
  p-value), post-fit data/model comparisons, corrected result vs theory/prior.
- **Conceptual and methodology diagrams** — for non-trivial multi-step
  procedures (correction chains, sample merging, region definitions, tagging),
  embed a schematic in the relevant section (not a standalone chapter). The
  **figure-scrolling test**: the physics story should be understandable by
  scrolling through figures alone; if a reader hits a non-trivial method with
  no visual explanation, a diagram is missing. Encouraged, not mandatory. See
  §D below.
- **Mandatory comparison overlay.** Results must contain at least one figure
  overlaying the measurement with published reference values on the SAME axes
  (not separate panels, not a table). Requirements: published points with
  uncertainties, this measurement with its total uncertainty band, a ratio or
  pull panel, and a chi2/ndf annotation (full covariance if available). If no
  published measurement exists at the exact points, use the closest and
  explain in the caption. No comparison overlay → Category B; saying
  "consistent with published values" without showing it → Category A.
- **Systematic breakdown figure.** The systematics section must contain a
  figure showing each source's relative contribution to the total (waterfall,
  horizontal bar, or stacked bar). A summary table alone is insufficient.

---

### Requirements

- Self-contained: all results inline, publication-quality figures.
- **Machine-readable results** in `results/` (CSV/JSON for spectra, covariance
  matrices).
- **Completeness test:** an unfamiliar physicist reproduces every number from
  the AN alone.
- **BibTeX:** `[@key]` with `references.bib`. Entries must include `doi`,
  `url`, `eprint`; use `unsrt`-style; use `get_paper` for RAG papers. **Never
  generate BibTeX from training data** — every entry must come from a
  verifiable source (`get_paper`, DOI lookup, INSPIRE search, or the actual
  PDF). Writing `@article{...}` from memory is almost certainly hallucination
  (fabricated INSPIRE key, authors, journal, even the paper's existence). A
  BibTeX entry not traceable to a real source is Category A.

### Literature requirements

- **Foundational citations (Category A if missing).** Cite the original
  theoretical papers defining the measured observable. "Mentioned by name but
  not cited" is Category A.
- **Published data overlay.** When prior measurements of the same observable
  exist at similar energy, Results must include a quantitative comparison
  figure (overlay or ratio/pull). "Consistent with published values" without
  showing it is Category B. If points aren't machine-readable, state so and
  give a table at representative bins.
- **Cross-experiment context.** The Introduction must cite at least 2
  measurements of the same/related observable from other experiments (LEP,
  Tevatron, LHC) where they exist. For a first measurement, cite closest
  precursors and explain what is new.
- **Reference count diagnostic.** **Fewer than 15 references in a 50+ page AN
  is Category A**; fewer than 20 is Category B. A thorough AN cites
  foundational theory (~3-5), reference analyses (~3-5), detector papers
  (~2-3), methodology (~3-5), and PDG/world-average sources (~3-5) — 15-25 is
  typical. The executor must verify count > 15 before typesetting.
- **Phase 1 data extraction (binding).** When reference analyses are
  identified in Phase 1, extract not just the systematic list but the
  **published numerical results** (central values + uncertainties at specific
  kinematic points) as quantitative comparison targets. These are binding at
  Phase 4c review (§6.8). Ad-hoc lookup at Phase 5 is too late — comparison
  data must be on disk before Phase 4a.

---

### LaTeX compilation

Markdown → PDF via a three-step pipeline: **pandoc** (>=3.0) produces `.tex`,
**`postprocess_tex.py`** applies deterministic structural fixes (title math,
escaped standalone math, margins, abstract→environment, references unnumbering,
table spacing, short longtable→table conversion, FloatBarrier, needspace,
duplicate headers/labels, appendix, clearpage, stale phase label warnings),
then **tectonic** (or xelatex) compiles to PDF. The `build-pdf` pixi task runs
this. The preamble (`conventions/preamble.tex`, symlinked from
`src/conventions/`) sets default figure **height** `0.45\linewidth`
(height-based so colorbar figures match plain-plot plot-area size), narrowed
caption width (75%), relaxed float placement (90% max/page), and widow/orphan
penalties. **Do not modify the preamble per-analysis** without a documented
reason. Do not use an LLM for LaTeX conversion.

### Pandoc pitfalls (mandatory rules)

- **Never use `$\pm$`, `$<$`, `$>$`, `$-$`, `$\sim$` as standalone math.** Use
  Unicode `±`, `<`, `>`, `−`, `~`. Bare single-symbol math confuses
  pandoc-crossref (`$-$0.0939` renders with visible dollar signs). Use the
  Unicode minus `−` (U+2212) for negatives outside math. Only use `$...$` for
  actual expressions (`$M_Z$`, `$\chi^2/ndf = 1.5$`).
- **YAML title field does not render LaTeX math** — `$\sqrt{s}$` renders
  literally. Use Unicode (`√s = 91.2 GeV`) or fix the literal string in
  `postprocess_tex.py`.
- **Never use `\mathrm{}` in figure captions or section headers** — pandoc's
  alt-text conversion produces "`\mathrm` allowed only in math mode" errors.
  Use plain subscripts (`$\sigma^0_{had}$`) or text.
- **Never put `@ref` cross-references inside `$...$` math.** `$\Gamma_Z$
  (@eq:width)` is fine; `$\Gamma_Z (@eq:width)$` mangles the reference.
- **Section headers must not contain complex LaTeX.** "Gamma-Z interference",
  not `$\gamma$-Z interference ($j_{had} = 0.14$)`.
- **The abstract must not be a numbered section.** Auto-fixed by
  `postprocess_tex.py` (converts to `abstract` environment before
  `\tableofcontents`).
- **References must be unnumbered.** Auto-fixed by `postprocess_tex.py`
  (`\section*{References}` + `\addcontentsline`).
- **Tables need spacing from preceding text.** Auto-fixed by
  `postprocess_tex.py` (`\vspace{1em}` before each `\begin{longtable}`).

### Table formatting

Pipe tables become `longtable`. To avoid overflow: **keep columns narrow**
(abbreviations, short headers; long descriptions to footnotes/prose); **avoid
monospace** (file paths overflow — use short labels + appendix lookup); **split
wide tables** (>6 columns → two tables or rotate); **consistent numeric
precision** (2-3 sig figs for uncertainties; don't typeset `91.17930000` when
`91.179` suffices); **test with `build-pdf`** (overfull hbox warnings indicate
overflow — fix before review).

### Rendering quality checklist

The executor (and the reviewer at the Phase 5 gate) must check these. All are
Category A:
- **Orphaned section headings** — a heading must not be the last line on a
  page. Use `\needspace{4\baselineskip}` before major sections; the preamble
  should set `\widowpenalty=10000` and `\clubpenalty=10000`.
- **Figures off-page** — any clipped figure must be resized; verify `width=`
  attributes. 2D plots with colorbars are prone to this (verify
  `make_square_add_cbar`).
- **Table splitting** — short tables (< 15 rows) must not split across pages.
  Use `\FloatBarrier`, position after first reference, or a `table` float.
- **Caption width** — captions span the full text width (default pandoc; do
  not override).
- **Consistent cross-references** — all `@fig:`/`@tbl:`/`@eq:` must resolve.
  Prefer `@sec:` labels over hardcoded section numbers.
- **Overfull hbox warnings** — investigate and resolve every one.
- **Abstract formatting** — before the TOC as an unnumbered block, not
  "Section 1: Abstract".
- **Table-caption collision** — separate captions from preceding paragraphs
  (`\vspace{1em}` before the table if needed).
- **Figure overflow (mandatory post-compilation check).** After compilation,
  run `grep "Overfull.*hbox" *.log`. Any overfull hbox involving a figure or
  table is Category A. Verify 2D-colorbar and multi-panel figures fit within
  `\linewidth`. Figures taller than `0.7\textheight` push captions to the next
  page — add explicit `height=`. Run `build-pdf` and check the log before
  review.

---

## Appendix D: Plotting Template

All plotting code must follow this template. **These rules are non-negotiable
and must be treated as gospel.** Any deviation requires an explicit, documented
justification in the experiment log — not a silent override. This is the
reference for any agent producing figures, executor or dedicated plotting
subagent.

### Base template

```python
import matplotlib.pyplot as plt
import mplhep as mh
import numpy as np

np.random.seed(42)            # if any randomness is involved
mh.style.use("CMS")

# --- Single plot ---
fig, ax = plt.subplots(figsize=(10, 10))

# ... plotting (mh.histplot for histograms; hist2dplot/cbarextend for 2D) ...

# Labels — required on EVERY independent axes, NEVER on ratio panels.
# Open data (this project): data=True with explicit llabel.
mh.label.exp_label(
    exp="<EXPERIMENT>",     # MANDATORY — e.g. "ALEPH", "CMS", "DELPHI"
    text="",                # e.g. "Preliminary" (leave "" for final)
    loc=0,
    data=True,              # always True for open data — control via llabel
    llabel="Open Data",     # "Open Data" for data, "Open Simulation" for MC
    rlabel=r"$\sqrt{s} = 91.2$ GeV",  # CMS style prints "TeV" for com=, so
                            # use rlabel for non-LHC sqrt(s)
    ax=ax,
)

fig.savefig("output.pdf", bbox_inches="tight", dpi=200, transparent=True)
fig.savefig("output.png", bbox_inches="tight", dpi=200, transparent=True)
plt.close(fig)
```

Ratio plot (single matplotlib figure — the only case where multiple panels
share one figure, because they need `sharex` + `hspace=0`):

```python
fig, (ax, rax) = plt.subplots(
    2, 1, figsize=(10, 10),
    gridspec_kw={"height_ratios": [3, 1]},
    sharex=True,                       # REQUIRED — else redundant x-label
)
fig.subplots_adjust(hspace=0)          # REQUIRED — no gap between panels
# ... plot main + ratio ...
ax.tick_params(labelbottom=False)      # avoid main/ratio tick collision
mh.label.exp_label(exp="ALEPH", data=True, llabel="Open Data", loc=0, ax=ax)
# Suppress spurious "Axis 0" text on ratio panel (some mplhep versions):
for txt in rax.texts:
    if "Axis" in txt.get_text():
        txt.remove()
```

### Rules

- **Style:** always `mh.style.use("CMS")` as base; branding comes from
  `exp_label`, not the style.
- **Font sizes are LOCKED.** Never pass numeric `fontsize=` to any matplotlib
  call — the CMS stylesheet sets sizes for the 10x10 figure. Relative strings
  (`'small'`, `'x-small'`, `'xx-small'`) are allowed (dense legends,
  annotations). Any numeric font size is Category A.
- **Legend font size:** always `ax.legend(fontsize="x-small")`.
- **Legend placement.** Default: scale the y-axis to fit the legend via
  `from mplhep.plot import mpl_magic; mpl_magic(ax)` after all plotting, with
  `loc="upper right"`. Use manual placement (`loc="center right"`,
  `"lower right"`, `bbox_to_anchor`) only when a genuinely empty region exists
  (ROC curves, exponential tails, log-scale plots). **Legend-data overlap is
  Category A** — the executor must visually inspect every figure, not
  just check `loc=`.
- **Publication-quality text.** Axis labels, legend entries, and tick labels
  must use human-readable names, not code identifiers. "Two-photon bkg", not
  "two_photon_bkg". Any raw code identifier in a rendered figure is Category A.
- **Aspect and colorbars (Category A if wrong).** Keep figures square. For ANY
  2D plot with a colorbar (`pcolormesh`, `imshow`, `hist2dplot`), use one of:
  `mh.hist2dplot(H, cbarextend=True)` (preferred); `cax =
  mh.utils.make_square_add_cbar(ax)` then `fig.colorbar(im, cax=cax)`; or `cax
  = mh.utils.append_axes(ax, extend=True)` then `fig.colorbar(im, cax=cax)`.
  **`fig.colorbar(im)`, `fig.colorbar(im, ax=ax)`, `fig.colorbar(im, ax=ax,
  shrink=...)`, and `plt.colorbar(...)` are Category A** — they steal space
  from the main axes. Applies to ALL 2D plots (correlation/migration/response
  matrices, 2D systematic maps, efficiency maps). Reviewers grep for these.
- **2D equal-aspect ratio.** For 2D histograms where both axes are the same
  coordinate type (two log-momenta on a Lund plane, two bin indices in a
  response matrix), set `ax.set_aspect('equal')` so bins render square. This is
  separate from `figsize=(10, 10)`. Axes with genuinely different units (mass
  vs angle) should NOT use equal aspect.
- **No titles.** Never `ax.set_title()` — captions go in the AN. Extra info via
  `ax.legend(title="...")` or, when necessary, `mh.label.add_text(text, ax=ax)`.
- **No raw `ax.text()` / `ax.annotate()`** on data plots — use
  `mh.label.add_text(text, ax=ax)` (including panel labels). (Exception:
  conceptual diagrams, below.)
- **Axis labels with units.** `r"$p_T$ [GeV]"` etc.; no `fontsize=`.
- **Labels on every independent axes.** Call `exp_label(...)` on EACH axes of
  multi-panel figures where each panel is independent (2x2 grids, side-by-side).
  **Exception: ratio plots** — `exp_label` on the MAIN panel ONLY, never on the
  ratio panel (Category A).
- **Open data labeling (mandatory).** Data: `exp="ALEPH", data=True,
  llabel="Open Data"` → "ALEPH Open Data". MC: `llabel="Open Simulation"` →
  "ALEPH Open Simulation". Replace ALEPH with the right experiment. Always
  `data=True` with explicit `llabel`. Labeling as just "ALEPH" without "Open
  Data"/"Open Simulation" implies an official result — Category A.
- **Label stacking pitfall (Category A).** `data=False` auto-adds "Simulation"
  as the left label; combining with `llabel`/`text` produces "Simulation Open
  Simulation". Safe pattern is always `data=True` with explicit `llabel`
  (`llabel=""` → just "ALEPH" for non-open-data). Never `data=False` with
  `llabel`.
- **Save as PDF and PNG**, always `bbox_inches="tight", dpi=200,
  transparent=True`.
- **Never `tight_layout()` or `constrained_layout=True`** with mplhep — they
  conflict with label positioning. Use `bbox_inches="tight"` at save time.
- **Close figures** — `plt.close(fig)` after saving.
- **Figure size is LOCKED at `figsize=(10, 10)`.** Non-negotiable — CMS font
  sizes are calibrated for it. Ratio plots: `(10, 10)` with
  `height_ratios=[3, 1]`. Grids: 10 inches per column, 10 per row (2x2 →
  `(20, 20)`, 1x3 → `(30, 10)`). Any custom figsize is Category A.
- **PDF rendering size (height-based).** The preamble sets default image
  **height** `0.45\linewidth` (height-based because colorbar figures have
  extended width; gives consistent plot-area sizing since all figures are
  square). `postprocess_tex.py` caps pandoc-wrapped figures at
  `height=0.35\textheight` to prevent overflow.

  | Plot type | matplotlib figsize | AN height |
  |-----------|-------------------|-----------|
  | Single panel | `(10, 10)` | `0.45\linewidth` (default) |
  | Ratio plot | `(10, 10)` | `0.45\linewidth` (default) |
  | 2D with colorbar | `(10, 10)` | `0.45\linewidth` (default) |
  | Side-by-side | Compose in LaTeX | `height=0.45\linewidth` each |
  | 2x2, 3x2 grid | Compose in LaTeX | `height=0.3\linewidth` each |

  **Prefer LaTeX subfigures over matplotlib grids.** Produce individual
  `(10, 10)` figures and compose in the AN. The one exception:
  tightly-coupled panels sharing a physical x-axis (ratio plots, pull
  distributions below a fit) — produce as a single figure with `sharex=True`,
  `hspace=0`. Panels with different x-axis variables go as separate `(10, 10)`
  figures composed via side-by-side `\includegraphics`. Test: if you cannot
  use `sharex=True`, produce separate outputs. Override default width with
  `![Caption](figures/name.pdf){#fig:name width=100%}`.
- **Ratio plot `sharex` and `hspace`.** Both `sharex=True` and
  `fig.subplots_adjust(hspace=0)` are non-negotiable. Missing `sharex=True`
  (redundant "Axis 0" x-label) and missing `hspace=0` (visible gap) are each
  Category A.
- **"Axis 0" artifact.** Some mplhep versions render spurious "Axis 0" on
  ratio panels when `exp_label(loc=0)` is on the main panel of a `sharex=True`
  figure. Workaround: after `exp_label`, remove it (see ratio snippet), or use
  `loc=2`. Any visible "Axis 0" text is Category B.
- **Ratio panel tick collision.** With `sharex=True` + `hspace=0`, the main
  panel's bottom y-tick and the ratio panel's top y-tick collide. Hide the
  main x-tick labels (`ax.tick_params(labelbottom=False)`) and set ratio
  y-limits/ticks so the top tick isn't at the boundary (e.g.,
  `rax.set_ylim(0.85, 1.15)`, ticks `[0.9, 1.0, 1.1]`). Overlapping boundary
  ticks are Category B.
- **Y-axis bin width labels.** Round bin widths to clean values (0.01, 0.05,
  0.1, 0.5, 1, 2, 5, 10...). If ugly (e.g., "Tracks / 0.04583"), adjust binning
  or omit the bin width ("Events"/"Tracks"). A non-round bin width is
  Category B.
- **Suppress matplotlib offset notation.** Never allow automatic "1e6"/"1e7"
  offset text. Use `ax.ticklabel_format(axis='y', style='plain')`, or absorb
  the multiplier into the label (`r"Tracks [$\times 10^6$]"` + divide data), or
  a custom major formatter. A raw offset string is Category B.
- **Log scale.** Use `ax.set_yscale("log")` when the y-range spans more than 2
  orders of magnitude; linear otherwise.
- **`mh.histplot()` for raw-count histograms (Category A if violated).** For
  histograms filled from raw event data (event/track counts, unweighted),
  use `mh.histplot()` — it computes sqrt(N) Poisson error bars. Never
  `ax.step()`, `ax.bar()`, or `ax.fill_between()` for these.

  | Data type | Correct | Wrong |
  |-----------|---------|-------|
  | Raw counts (error bars) | `mh.histplot(h, histtype="errorbar")` | `ax.errorbar` without yerr |
  | MC prediction (filled) | `mh.histplot(h, histtype="fill")` | `ax.fill_between()`, `ax.bar()` |
  | Stacked MC | `mh.histplot([h1,h2], stack=True)` | `ax.bar(..., bottom=...)` |
  | Theory curve | `ax.plot(x, y)` | (correct — not binned) |

- **Derived quantities MUST pass explicit `yerr` (Category A if violated).**
  Any quantity that is NOT a raw count — normalized distributions, correction
  factors, EEC values, ratios, efficiencies, systematic shifts — MUST pass
  explicit `yerr=`. Without it, mplhep applies sqrt(bin content), which is
  meaningless (an EEC value of 0.03 gets sqrt(0.03)=0.17, a 570% error). Use
  either `ax.errorbar()` or `mh.histplot()` — both accept `yerr=`. **Test:**
  filled with `h.fill(raw_values)` → `mh.histplot` without `yerr` is correct;
  assigned via `h.view()[:] = ...` or computed from a formula → MUST pass
  `yerr=`. For step aesthetics without error bars, use
  `mh.histplot(h, histtype="step")`. Exception: weighted histograms filled
  with `h.fill(values, weight=weights)` using `Weight()` storage have correct
  auto-errors.
- **Deterministic** — `np.random.seed(42)` if any randomness is involved.
- **Ratio panel uncertainty bands** — any shaded band must be described in the
  legend or caption (unexplained band is Category B). Suspiciously flat
  (constant-width) bands warrant verifying the error computation.
- **Axis limits** — set tight to the data; no large empty regions
  (`set_xlim`/`set_ylim` when auto-range adds whitespace). Log y-axes start at
  ~0.5x the minimum non-zero value.
- **Sanity check: visually identical distributions.** If two distributions
  that should be independent (different years, different systematic variations,
  different observables) look indistinguishable, flag for investigation — could
  be a bug (same array plotted twice), a tautological comparison, or genuine
  high correlation. Do not silently accept.

### Pre-commit figure verification (executor responsibility)

Before committing any plotting script, the executor MUST self-verify every
figure (the reviewer is a safety net, not the primary check). A figure
with `ax.step()` for histogram data or `cos_th` in a legend is the executor's
responsibility.

**Step 1: Code grep.** Flag any of: `ax.step(`, `ax.bar(` (unless
colorbar-related), `ax.text(`, `tight_layout`, `plt.colorbar`, `set_title(`,
`fig.colorbar(.*, ax=` (use `cax=`), `fontsize=\d` (absolute font). Detect the
derived-quantity trap: `.view()[` present with `histtype="errorbar"` but no
`yerr=` (mplhep applies sqrt(N) to non-count values). Require positive
patterns: any `pcolormesh`/`imshow`/`hist2dplot` must have
`make_square_add_cbar` or `cbarextend`; any `ax.legend(` should have
`mpl_magic`; any `sharex=True` + `exp_label` must suppress "Axis 0" (`Axis`
remove loop or `loc=2`).

**Step 2: Visual inspection of every rendered PNG** — at actual size:
- [ ] Legend does not overlap data, error bars, or curves
- [ ] All text (axis/tick/legend) is readable
- [ ] No duplicate experiment labels (main panel only, never on ratio)
- [ ] Variable names are publication-quality (no `_` outside LaTeX math)
- [ ] Histograms use `histplot` step/errorbar style (not smooth lines)
- [ ] Axis ranges tight to the data
- [ ] 2D colorbars are the same height as the main axes
- [ ] 2D bins appear square when both axes are the same coordinate type
- [ ] Figure contains actual data (not empty axes/placeholder/white canvas)
- [ ] **Error bar sanity** — derived quantities have plausible error bars
  (few percent to tens of percent, not larger than the signal). Error bars
  spanning the full y-axis or visibly >100% mean `yerr` was not passed.
  Category A.

**Step 3: Label quality grep** — check `label=`, `set_xlabel`, `set_ylabel`
strings for code variable names (an `_` outside `$...$` math is a leaked
identifier). Fix all Step 1-3 violations before committing.

### Error propagation for derived quantities

Uncertainties for ratios, normalized distributions, and efficiencies must be
propagated manually (matplotlib does not):
- **Normalized** `(1/N) dN/dx`: `yerr[i] = sqrt(n[i]) / (N * dx[i])`,
  `N = sum(n)`.
- **Ratio** `R = A/B`: `sigma_R = R * sqrt((sigma_A/A)^2 + (sigma_B/B)^2)`
  (uncorrelated).
- **Efficiency** `epsilon = k/n`: Clopper-Pearson (binomial) intervals via
  `scipy.stats.binom`, not Gaussian.
- **Bin-width-normalized** `dN/dx`: `yerr[i] = sqrt(n[i]) / dx[i]`.

Always pass `yerr=` explicitly for derived quantities; never rely on mplhep
auto-errors for anything other than raw counts or properly weighted
histograms. Omitting `yerr=` on derived quantities is Category A.

### Captions

See §5.2 for caption requirements. Captions must be self-contained:
**`<What> (<where>) <description/conclusion>.`** — name the observable; for
composites indicate the panel ("(left)", "(top left)"); state context not
visible in the plot (selection, normalization, which phase/systematic this
validates) and the key conclusion. A good caption is 2-4 sentences and:
1. names the plot (observable, selection stage, comparison);
2. for composites, identifies each panel with a positional pointer;
3. states context not visible in the plot;
4. states the key observation/conclusion.

**Do NOT restate the legend or axis labels.** Captions under two full sentences
are Category A.

Bad (Category A, too sparse): "Thrust distribution." Good: "Charged hadron
multiplicity after the hadronic event selection, normalized to equal area. The
mean multiplicity of ~20 is characteristic of hadronic Z decays. Data/MC
agreement is within 2% across the full range; the low-multiplicity tail is
sensitive to two-photon background contamination (see §5.3)."

### Figure compositing in the AN

Group related figures side-by-side rather than as separate full-page figures.
**Do NOT use `\subfloat` or `\subcaptionbox`** (clunky layouts, redundant
sub-captions, excess whitespace). Use simple side-by-side `\includegraphics`
with a single unified caption.

Pair:
```latex
\begin{figure}[!htbp]
\centering
\includegraphics[height=0.38\linewidth]{figures/left.pdf}\hspace{1em}
\includegraphics[height=0.38\linewidth]{figures/right.pdf}
\caption{The ln(1/Delta-theta) projection (left) and ln(kt) projection
  (right) of the unfolded Lund plane density, compared to Pythia 8 Monash
  and Pythia 6. Both projections show ...}
\label{fig:label}
\end{figure}
```

2x2 grid:
```latex
\begin{figure}[!htbp]
\centering
\includegraphics[height=0.32\linewidth]{figures/tl.pdf}\hspace{1em}
\includegraphics[height=0.32\linewidth]{figures/tr.pdf}\\[0.5em]
\includegraphics[height=0.32\linewidth]{figures/bl.pdf}\hspace{1em}
\includegraphics[height=0.32\linewidth]{figures/br.pdf}
\caption{Shift maps for four sub-dominant sources: angular resolution
  (top left), non-closure (top right), thrust axis (bottom left), and
  momentum resolution (bottom right). All are below 1\% ...}
\label{fig:label}
\end{figure}
```

**Key rules:** use `height=` (not `width=`) so different aspect ratios align;
minimum panel height `0.30\linewidth` (below this text is illegible); no
`(a)`/`(b)` labels — use "(left)"/"(right)" in caption text; the caption is a
single paragraph synthesizing all panels following `<What> (<where>)
<description>`.

**All compositing MUST be done in LaTeX.** Never use PIL, matplotlib grid
stitching, or any external image tool — these produce inconsistent DPI, wrong
font sizes, and artifacts. If LaTeX compositing cannot achieve the layout,
simplify the layout, don't reach for an external tool.

**Visual appeal check (mandatory for composites).** After compositing, inspect
the rendered PDF: are all panels the same size (different types at the same
`height=` produce different sizes — adjust per-panel or group same-type
panels)? Is the grid symmetric and centered with no ragged edges and uniform
small gaps? Would a reviewer find it professional? If it looks haphazardly
pasted, redo it.

**Sizing reference:**

| Grid | Per-panel height | Use case |
|------|-----------------|----------|
| 1x2 (pair) | `0.38\linewidth` | Projection comparisons, matrix pairs |
| 2x2 | `0.32\linewidth` | Sub-dominant shift maps |
| 1x3 (row) | `0.30\linewidth` | Simple data/MC surveys |
| 2x3 | `0.30\linewidth` | Per-subperiod or per-cut comparisons |
| 3x3 | `0.28\linewidth` | Full input variable survey |

**Data/MC comparison surveys** (one main + one ratio panel) are highly
compressible — use **3 columns** spanning the full page at `0.30\linewidth`
height. A 7-plot survey becomes 3+3+1 or 4+3; a 5-plot survey becomes 3+2. Do
NOT use 2-column layouts for simple data/MC. When presenting N related
distributions (input variables, per-cut motivation, per-systematic shift maps,
per-subperiod), compose them as a single multi-panel figure.

### Figure cross-referencing

Use pandoc-crossref syntax:
- **Label every figure:** `![Caption text](figures/name.pdf){#fig:name}`.
- **Reference:** `@fig:name` (produces "fig. X"); at sentence start `Figure
  @fig:name`.
- **Never use** `[-@fig:...]` — always `@fig:name`.
- Tables use `{#tbl:name}` / `@tbl:name`; equations `{#eq:name}` / `@eq:name`.

### Correlation and covariance visualizations

Show correlations between variables, bins, or systematic sources as **matrix
heatmaps** (`mh.hist2dplot` or `ax.pcolormesh` with a diverging colormap like
`RdBu_r`, centered at 0). Never as overlaid 1D distributions or scatter grids
(unreadable for more than ~3 variables). For the correlation matrix: `vmin=-1,
vmax=1`; annotate cells if small (< 10x10); otherwise show the heatmap without
annotations but with a clear colorbar.

### Systematic breakdown plots

When a breakdown shows any single source >100% relative uncertainty in a bin,
investigate — usually a bug in variation processing or a low-stats edge effect.
Clip or flag such bins rather than letting them dominate the y-axis. If
genuine, document the explanation in the artifact.

### Delegation to plotting subagent

Plotting may be delegated to a dedicated subagent. The parent prompt must
include: (1) this entire appendix (template + rules); (2) the data to plot
(paths or serialized arrays); (3) the plot kind (histogram, ratio, 2D,
overlay); (4) axis labels and ranges; (5) experiment-label parameters (exp,
com, lumi, data flag); (6) output path. The plotting agent applies this
template — it does not make physics decisions about what to plot or how to
interpret.

### Conceptual and schematic diagrams

Conceptual diagrams (analysis flow, correction chains, sample composition,
region definitions) fill gaps where data plots don't convey methodology. They
are embedded in the relevant AN section, not a standalone chapter (a
correction-chain diagram in Corrections; a region-definition diagram in Event
Selection). **The figure-scrolling test:** the whole physics story should be
understandable by scrolling through figures alone; a non-trivial method with no
visual explanation means a diagram is missing.

**Production rules:** `mh.style.use("CMS")` for consistency; `exp_label` NOT
required (not data plots); `figsize=(10, 10)` does NOT apply (use whatever fits
— wide flow charts, tall chains); build with matplotlib patches, arrows,
`FancyBboxPatch`, `FancyArrowPatch`, and `ax.text()` (exception to the
`ax.text` rule — schematic elements, `mh.label.add_text` is not appropriate);
save PDF + PNG (`bbox_inches="tight", dpi=200, transparent=True`); captions 2+
sentences in the standard format.

**Common types:** analysis flow (stage boxes, data-flow arrows, signal/control
branching); sample composition (stacked/grouped boxes per selection stage);
correction chain (corrections with inputs/outputs per step); region definitions
(signal/control/validation regions in discriminant space).

Conceptual diagrams are encouraged, not mandatory — identified in Phase 1,
produced in Phase 5. The figure-scrolling test is the quality criterion.
