# Appendix

Consolidated reference: orchestration automation, phase graph, RAG integration, agent roles, and session/directory layout. Architectural rationale and phase definitions are in `02-phases.md` (§3a, §3); the review protocol in `03-review.md`.

## Automation (orchestration logic)

Pseudocode, not runnable — helpers (`find_latest_artifact`, `extract_decision`, `present_for_human_review`) are orchestrator responsibilities. One executor call per phase (stats + AN + typeset in one role) and one review function spawning a single critical reviewer (no arbiter, no panel).

```
for phase in [1, 2, 3, 4a, 4b, 4c, 5]:
    run_executor(phase)                 # plan → code → artifact (+ AN+PDF from 4a)
    if phase has a reviewer:            # lean default: 1, 3, 4a  (4b→human gate; 4c/5→orchestrator checklist)
        loop:
            run_critical_reviewer(phase)
            if PASS: run_regression_check(phase); break
            else (ITERATE): run_fixer(phase)   # clear Category A; escalate to human if stuck
    commit(phase)
    if phase == 4b: present_PDF_to_human(); wait_for_decision()   # APPROVE/ITERATE/REGRESS/PAUSE
```

`run_regression_check`: if the review (or the orchestrator's own checklist) flags a §6.7 trigger, log it, spawn the Investigator (→ `REGRESSION_TICKET.md`), spawn a fixer on the origin phase, re-review, re-run downstream. Phase 2 is self-review (no reviewer call). Multi-channel: Phases 2–3 may run per-channel in parallel, reviewed sequentially, then consolidated; shared calibrations finish before 4a.

## Phase dependency graph

```
Physics Prompt → Ph1 Strategy → Ph2 Exploration → Ph3 Selection (per channel) →
  Ph4a Expected (review) → Ph4b 10% (review → HUMAN GATE) → Ph4c Full → Ph5 Documentation
            ▲ Experiment Corpus (RAG) queried throughout
            └ phase regression re-enters at the origin phase if a fundamental issue is found
```

Both analysis types share this. For searches, 4b is a partial unblinding (10% of SR data); for measurements, a 10%-vs-expected consistency check. The human gate sits between 4b and 4c in both.

## RAG integration
A SciTreeRAG corpus may be available to all sessions via MCP (no per-session setup). Agents query as needed and cite sources (paper ID + section); failed retrievals go to `retrieval_log.md` (§2.2). Tools: `search_lep_corpus(query, top_k, experiment, mode)` (hybrid dense+BM25 retrieval; prefer `mode="hybrid"`), `get_paper(id)` (drill into a reference), `list_corpus_papers(...)`, `compare_measurements(topic)` (cross-experiment). No RAG → proceed on `docs/` + training knowledge, marking uncorroborated claims unverified. **Tool heuristics** (e.g. mplhep: use the generic `mplhep.label.exp_label`, never experiment-specific `mplhep.cms.label` which adds branding; always set `rlabel` explicitly; with `data=False` mplhep auto-adds "Simulation") are recorded as learned, consulted before re-querying docs; full plotting rules in `04-output.md`.

## Agent roles

Full definitions in `../agents/*.md` (index in `../agents/README.md`). Context assembly follows §3a.4 (bird's-eye framing + relevant methodology sections + upstream artifacts); phase CLAUDE.md files (from `../templates/`) are what agents read at runtime. All subagents run `model: "opus"` and read files with the Read tool (no `cat`/`sed`/`head`).

| Role | Definition | Writes |
|------|-----------|--------|
| Executor | `agents/executor.md` | `outputs/` artifacts, `src/` code, `outputs/figures/`, `ANALYSIS_NOTE_{phase}_v{N}.{md,tex,pdf}` (4a+), `logs/` |
| Fixer | `agents/fixer.md` | updated artifact + code, `logs/` (replaces the executor on ITERATE) |
| Critical reviewer | `agents/critical_reviewer.md` | `review/critical/`, `logs/` (single reviewer; PASS/ITERATE; only Category A blocks) |
| Investigator | `agents/investigator.md` | `REGRESSION_TICKET.md`, `logs/` (regression diagnosis only) |

The executor is one role doing statistics + AN prose + typesetting (pandoc → `postprocess_tex.py` → tectonic; compile→read→fix). PDF compilation is mandatory before review at 4a and the 4b human gate (the reviewer/human reads the PDF). Mechanical figure/BibTeX lint is the executor's self-lint, not a separate reviewer.

## Sessions and layout

Every agent invocation is an isolated session — reads files, writes files, exits; files are the interface (no shared history/memory). The **exception** is `experiment_log.md`: one per phase (or per parallel track), append-only, the only mutable shared state across sequential sessions in that track. Each agent gets a unique **session name** (a random human first name, not reused within a run) used in output filenames so iteration creates new files rather than overwriting; downstream agents pick the most recent by timestamp. Each agent maintains an **incremental session log** in `logs/{role}_{name}_{timestamp}.md` — written progressively (crash-resilient), short timestamped milestone entries (session start, plan, code, test, figure, decision, error, artifact, end), not a transcript.

Directory layout (one representative phase shown; others follow the same pattern; Phase 2 has no `review/`; AN phases also hold `INFERENCE_*.md` + `ANALYSIS_NOTE_*.{md,tex,pdf}`):

```
analysis_name/
  prompt.md  regression_log.md
  calibrations/<name>/        # shared sub-analyses (own log + CALIBRATION_<NAME>.md + src/outputs)
  phase1_strategy/
    experiment_log.md  retrieval_log.md  src/  outputs/{figures,STRATEGY.md,plan.md}
    review/critical/   logs/
  phase2_exploration/         # self-review — no review/
  phase3_selection/           # per-channel subdirs if multi-channel; SELECTION_COMBINED.md to consolidate
  phase4_inference/{4a_expected,4b_partial,4c_observed}/   # each: outputs + review/critical (4b → human)
  phase5_documentation/       # outputs/ANALYSIS_NOTE_5_v1.{md,tex,pdf} + review/critical
```
