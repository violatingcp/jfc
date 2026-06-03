# Phase 3: Selection

Implement the Phase-1 approach for a **{{analysis_type}}** analysis (read `STRATEGY.md` first).
**Start in plan mode** (scripts, selection, figures), then execute.

## Output artifact
`outputs/SELECTION.md` — object/event selection, cutflow, approach comparison, validation. Save per-sample selected events for Phase 4.

## Applicable conventions
Identify the technique from `STRATEGY.md`, open the matching convention(s) ({{conventions_files}}), and implement every required step — omissions are Category A.

## Key requirements
- **≥2 selection approaches** evaluated on a common figure of merit; choose one and justify (cut-based instead of MVA is a documented downscope). If MVA: ROC + score (train/test) + feature importance + data/MC of the score.
- **Every cut motivated by a plot** (N-1 preferred). Cutflow monotonically non-increasing.
- **Background model closes:** data/MC comparison for variables entering the selection; report a closure χ²/p-value. If a validation/closure test fails, attempt ≥3 documented remediations before writing the artifact.
- **Blinding** (searches/extractions): optimize on MC expected sensitivity, never on observed signal-region data.
- Analysis and plotting code separated; figures pass `pixi run lint-plots`.

## Closure alarm bands (Category A)
χ²/ndf < 0.1 (suspicious) · χ²/ndf > 3 or pull > 5σ (failure) · `passes:false` in JSON while text claims OK. Don't frame failures as "known limitations".

## Self-check
- [ ] ≥2 approaches compared; every cut has a plot; cutflow monotonic.
- [ ] Closure passes (or ≥3 remediations); convention requirements met; lint clean.

## Review
Single critical reviewer — PASS / ITERATE; only Category A blocks. Findings to `review/critical/`.
