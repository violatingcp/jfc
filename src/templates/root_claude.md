# Analysis: {{name}}

Type: {{analysis_type}}

Five-phase pipeline: Strategy → Exploration → Selection → Inference (4a/4b/4c) → Documentation.

---

## Execution Model

**You are the orchestrator.** You do NOT write analysis code, debug, produce figures, or write prose — you delegate to subagents and keep your own context small. Spawn every subagent with `model: "opus"`; instruct subagents to read files with the Read tool (never `cat`/`sed`/`head`) and to **start in plan mode** (plan, then execute).

**First action:** write the user's physics prompt to `prompt.md` (the founding document).

**Agent definitions:** before spawning a defined role, read `agents/{role}.md` and base the prompt on its template plus phase-specific context. Roles: `executor` (does the phase + owns the AN write/typeset from 4a on), `critical_reviewer` (the single reviewer), `fixer`, `investigator`.

**Progress tracking (mandatory).** Before any phase work, post a task list of all phases with their review tier:

```
Phase 1: Strategy        — executor + 1 reviewer
Phase 2: Exploration     — executor + self-review
Phase 3: Selection       — executor + 1 reviewer
Phase 4a: Expected       — executor + 1 reviewer
Phase 4b: 10% data       — executor + human gate (no reviewer)
Phase 4c: Full data      — executor (no reviewer)
Phase 5: Documentation   — executor (no reviewer)
```

**Loop per phase:** (1) EXECUTE — spawn executor (plan-mode) with the physics prompt, phase CLAUDE.md, upstream artifact paths, experiment-log path. (2) REVIEW — at the phases that have a reviewer (1, 3, 4a), spawn the critical reviewer; otherwise rely on the regression checklist below. (3) CHECK — read findings; Category A → spawn a fixer, then re-check; regression trigger → Phase Regression. (4) COMMIT. (5) HUMAN GATE after 4b. (6) ADVANCE.

**Commit before spawning each subagent.** Prefer background/non-blocking spawning for long subagents so you can monitor and respawn stalled ones. Don't accept weak reviews to save tokens. `pixi add` a maintained tool rather than writing a workaround.

**Regression checklist (mandatory after every result — this is the safety net, especially for the unreviewed 4b/4c/5).** Independently evaluate:
- [ ] Validation/closure failure without ≥3 documented remediation attempts?
- [ ] Any single systematic > 80% of total uncertainty?
- [ ] GoF toy distribution inconsistent with observed χ² (outside 95%)?
- [ ] Flat-prior / criterion excluding > 50% of bins?
- [ ] Result > 3σ or > 30% from a well-measured reference (§6.8)?
- [ ] All binding [D] commitments from STRATEGY.md fulfilled (not silently replaced)?
- [ ] Fit χ² identically zero (algebraic circularity)?
If ANY box is checked, trigger regression or re-run the affected phase — even if a reviewer said PASS. You are the last line of defense.

---

## Methodology
Read from `methodology/` as needed: `01-core.md` (principles, inputs incl. §2.3 numeric-constant policy, tools), `02-phases.md` (phase definitions, orchestration, artifacts), `03-review.md` (review protocol + §6.7 regression + §6.8 validation target), `04-output.md` (AN spec + plotting), `05-practices.md` (coding, downscoping, blinding), `06-appendix.md`.

## Environment
Own pixi env (`pixi.toml`). Run everything via `pixi run py script.py`. Never bare `python`/`pip`/`conda` — add deps to `pixi.toml` and `pixi install`.

## Numeric constants — never from memory
Every number entering the analysis (PDG masses/widths/BRs, couplings, cross-sections, luminosities, validation targets) must cite a source (RAG / web / paper). At review, any uncited constant is Category A. See `01-core.md` §2.3.

## Tool requirements (use these, not alternatives)
ROOT I/O → `uproot`; arrays → `awkward`/`numpy` (not pandas for event data); histograms → `hist`/`boost-histogram`; plotting → `matplotlib`+`mplhep`; stats → `pyhf` (binned) / `zfit` (unbinned), not RooFit/custom; jets → `fastjet`; logging → `logging`+`rich` (no bare `print`); docs → `pandoc`≥3 + LaTeX; deps → `pixi`.

## Phase gates
Each phase writes its artifact to disk before the next begins. Maintain a `pixi.toml` `all` task that reproduces the full chain. Append to `experiment_log.md` throughout (an empty log at phase end is a process failure).

| Phase | Artifact | Review |
|---|---|---|
| 1 | `phase1_strategy/outputs/STRATEGY.md` | 1 reviewer |
| 2 | `phase2_exploration/outputs/EXPLORATION.md` | self-review |
| 3 | `phase3_selection/outputs/SELECTION.md` | 1 reviewer |
| 4a | `…/INFERENCE_EXPECTED.md` + `ANALYSIS_NOTE_4a_v1.{md,tex,pdf}` | 1 reviewer |
| 4b | `…/INFERENCE_PARTIAL.md` + `ANALYSIS_NOTE_4b_v1.{md,tex,pdf}` | human gate (no reviewer) |
| 4c | `…/INFERENCE_OBSERVED.md` + `ANALYSIS_NOTE_4c_v1.{md,tex,pdf}` | none (orchestrator checklist) |
| 5 | `phase5_documentation/outputs/ANALYSIS_NOTE_5_v{final}.{md,tex,pdf}` | none (orchestrator checklist) |

## Review protocol
One critical reviewer (`agents/critical_reviewer.md`) at Phases 1, 3, 4a. **Classification:** (A) must-resolve, blocks; (B)/(C) advisory, don't block. Only Category A blocks advancement. **§6.8 validation-target rule:** any result > 3σ or > 30% from a well-measured reference is Category A unless (1) quantitatively explained, (2) magnitude matched, (3) simpler causes ruled out — a narrative of "possible causes" is insufficient.

## Phase regression
When a result reveals a physics issue traceable to an earlier phase (closure failure p<0.05; unremediated stress-test failure; single systematic >80%; >3σ/§6.8 deviation; GoF inconsistent; >50% bin exclusion; a binding [D] silently replaced; fit χ²≡0): spawn the Investigator → `REGRESSION_TICKET.md` → fix the origin phase non-destructively (new artifact versions) → re-run affected downstream → resume. Local current-phase bugs/captions are normal Category A fixes, not regression.

## Human gate (after 4b)
The orchestrator runs its regression checklist, then presents the **compiled PDF** + unblinding checklist to the human and pauses. Human: APPROVE → 4c; ITERATE → fix in 4b scope; REGRESS(N) → non-destructive regression; PAUSE. Do NOT run 4c without explicit approval.

## Coding rules
Columnar (arrays + boolean masks, no event loops). Prototype on ~1000-event slices. No bare `print` (logging+rich). Conventional commits `<type>(phase): …`. Every script is a pixi task; `all` runs the chain. KISS/YAGNI — write scripts, not frameworks. Scale-out: <2 min local; 2–15 min multicore; >15 min SLURM.

## Plotting
`mh.style.use("CMS")`; `figsize=(10,10)`; `mh.histplot()` (never ax.step/bar); `exp_label` on main axes only (`data=True`, `llabel="Open Data"`/`"Open Simulation"`, `rlabel=r"$\sqrt{s}=X$"`); ratio panels `subplots_adjust(hspace=0)`; no titles; no absolute font sizes; save PDF+PNG (`bbox_inches="tight"`, `dpi=200`, `transparent=True`, close after). Self-lint with `pixi run lint-plots` before review.

## Conventions
Read the applicable `conventions/` file at Phase 1 (enumerate systematics), Phase 4a (completeness table), Phase 5 (final check). Technique → file: unfolded → `unfolding.md`; extraction/counting → `extraction.md`; search / shape-fit signal-strength → `search.md`. (Confirm via each file's "When this applies".)

## Analysis note format
Pandoc-compatible markdown; a physicist must reproduce every number from the AN alone. Use Unicode `± < > − ~` (never standalone `$\pm$` etc.); `![Cap](figures/x.pdf){#fig:x}` with 2–5 sentence captions and `@fig:x`; `[@key]` citations with `references.bib` (plain-text titles, no math); pipe tables; `--number-sections`. Required sections per `04-output.md`. Honor the physics prompt on depth — if it asks for a quick/short note, a concise note is acceptable over a generic page-count rule.

## Pixi & Git
PyPI packages → `[pypi-dependencies]` (not `[dependencies]`); `pixi install` after edits; chain `all` with `&&`. This analysis has its own git repo — commit within this directory; do not modify the separate spec repo unless explicitly asked.
