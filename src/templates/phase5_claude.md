# Phase 5: Documentation

Produce the final analysis note for a **{{analysis_type}}** analysis.
**Start in plan mode** (gaps in the current AN, figures to add, sections to polish), then execute. The AN already exists from Phase 4c — polish it, do not rewrite.

## Output artifacts
`outputs/ANALYSIS_NOTE_5_v1.{md,tex,pdf}` (pandoc markdown → typeset PDF) and machine-readable `results/`.

## Steps
1. **Figures:** aggregate/symlink the phase figures into `outputs/figures/`; add any missing AN-specific figure; verify every `figures/*.pdf` reference resolves (a missing figure is Category A).
2. **Polish the AN** (from `ANALYSIS_NOTE_4c_v1.md`): every section has intro prose; per-systematic subsections in running prose; ≥4 labeled equations; a validation summary table; a resolving-power statement; a comparison-with-published figure; a systematic-breakdown figure. Numbers quoted from `results/*.json` (>1% mismatch = Category A). No local filesystem paths.
3. **Typeset:** pandoc (`--standalone --number-sections --toc --filter pandoc-crossref --citeproc --bibliography=references.bib`, include `preamble.tex`) → `postprocess_tex.py` → compile (tectonic). Iterate compile→read→fix. PDF must have no `??` / `[?]`, figures render, captions ≥2 sentences.

## Markdown rules
Unicode `± < > − ~` (never standalone `$\pm$` etc.); `![Cap](figures/x.pdf){#fig:x}` with `@fig:x`; `[@key]` citations with `references.bib` (≥15 unique keys); state the luminosity; `# Change Log {-}`.

## Pre-finish gate
`pixi run all` reproduces the full chain; PDF compiles with all figures.

## Review
**No reviewer gate.** Self-lint + PDF compilation cover the mechanical checks; the orchestrator's regression + AN-completeness checklist runs after the note is produced and is the safety net. If it surfaces a Category A issue, spawn a fixer and re-check.

> Note on length: the default target is a thorough note. If the physics prompt asks for a quick/short analysis, a concise note is acceptable — honor the prompt over a generic page-count rule.
