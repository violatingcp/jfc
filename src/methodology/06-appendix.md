# Appendix

Consolidated reference material: orchestration automation, the phase
dependency graph, tool heuristics, agent-system integration, prompt-template
summaries, and session/directory layouts.

- Architectural rationale: `02-phases.md` (§3a orchestration).
- Phase definitions and gates: `02-phases.md` (§3).
- Review tiers and protocol: `03-review.md`.

---

## Appendix: Automation

Pseudocode for the orchestration logic. **Not a runnable script** — helpers
like `find_latest_artifact`, `extract_decision`, and `present_for_human_review`
are orchestrator responsibilities whose implementation depends on the agent
system. The logic and control flow are what matter.

The pipeline uses a **single executor call per phase** (the executor does
stats, AN writing, and typesetting in one role) and the review-tier functions
`run_2bot_review`, `run_1bot_review`, `run_1bot_review_with_bibtex`, and
`run_phase5_review`.

```bash
# --- Configuration ---
# Hard cap on review iterations. Correctness (arbiter PASS or no Category A
# finding) is the real termination condition; this prevents infinite loops.
max_review_iterations=${MAX_REVIEW_ITER:-10}

# --- Session naming ---
# Picks an unused random human first name from the pool, without replacement
# within a run. Any unique-per-run scheme works.
pick_session_name() { echo "$(shuf -n1 names_pool.txt)"; }

# --- Regression detection and upstream feedback ---
# Checks review output for regression triggers; if found, dispatches
# investigation + fix, re-reviews the origin phase at its tier, re-runs
# downstream. Returns non-zero so the caller does not proceed.
run_regression_check() {
  dir=$1
  review_artifact=$(find_latest_review_artifact "$dir")
  if grep -q "regression trigger" "$review_artifact"; then
    origin_phase=$(extract_regression_origin "$review_artifact")
    echo "$(date): $dir -> $origin_phase" >> regression_log.md
    run_agent --name "$(pick_session_name)" \
      --output "$origin_phase/REGRESSION_TICKET.md" \
      "Investigate regression trigger from $dir."
    run_agent --role fixer --name "$(pick_session_name)" \
      --output "$origin_phase/outputs" \
      "Fix regression described in $origin_phase/REGRESSION_TICKET.md"
    tier=$(get_review_tier "$origin_phase")
    if [ "$tier" = "2bot" ]; then run_2bot_review "$origin_phase"
    else run_1bot_review "$origin_phase"; fi
    rerun_downstream_from "$origin_phase"
    return 1
  fi
  return 0
}

check_upstream_feedback() {
  dir=$1
  # If $dir/UPSTREAM_FEEDBACK.md exists, reviewers discover it when reading
  # the phase contents at the next review gate. No routing action needed.
  [ -f "$dir/UPSTREAM_FEEDBACK.md" ] && echo "Upstream feedback in $dir"
}

# --- Review tiers (return 0 PASS, 1 regression, 2 escalation/max-iter) ---
#
# Three tiers run parallel reviewers, then an arbiter that emits a decision:
# run_2bot_review, run_phase5_review, run_1bot_review_with_bibtex. They share
# the loop below (`_arbiter_loop`), differing only in which reviewers run in
# parallel ($1 = the reviewer-spawning function). run_1bot_review has NO
# arbiter (it gates on Category A/B directly) and is written out separately.

_arbiter_loop() {   # $1 = function that spawns the parallel reviewers for $dir
  spawn_reviewers=$1; dir=$2; i=0
  while [ $i -lt $max_review_iterations ]; do
    i=$((i + 1))
    [ $i -gt 3 ] && echo "WARNING: review iteration $i for $dir"
    [ $i -gt 5 ] && echo "STRONG WARNING: review iteration $i for $dir"
    "$spawn_reviewers" "$dir"; wait     # reviewers run in parallel
    run_agent --name "$(pick_session_name)" \
      --output "$dir/review/arbiter" "arbitrate"
    case "$(extract_decision "$dir/review/arbiter")" in
      PASS)
        run_regression_check "$dir" || return 1
        check_upstream_feedback "$dir"; return 0 ;;
      ITERATE)
        # New session-named inputs (arbiter assessment + Category A issues +
        # original upstream artifacts). No overwrites — session naming keeps
        # each iteration's inputs/outputs coexisting on disk.
        exec_name=$(pick_session_name)
        write_iteration_inputs "$dir" "$i" "$exec_name"
        run_agent --role fixer --name "$exec_name" \
          --output "$dir/outputs" "fix iteration v$((i+1))" ;;
      ESCALATE) present_for_human_review "$dir"; wait_for_human_input ;;
    esac
  done
  echo "ERROR: review hit $max_review_iterations iterations for $dir"
  present_for_human_review "$dir"; wait_for_human_input; return 2
}

# Phase 1: physics (+ plot validator at figure phases) → arbiter.
# NOTE: omit plot_validator at Phase 1 (no figures).
_2bot_reviewers() {
  run_agent --role physics_reviewer --name "$(pick_session_name)" \
    --output "$1/review/physics" "physics review" &
  run_agent --role plot_validator --name "$(pick_session_name)" \
    --output "$1/review/validation" "plot validation" &
}
run_2bot_review() { _arbiter_loop _2bot_reviewers "$1"; }

# Phase 5: rendering + bibtex + plot validator → arbiter.
_phase5_reviewers() {
  run_agent --role plot_validator --name "$(pick_session_name)" \
    --output "$1/review/validation" "plot validation" &
  run_agent --role rendering_reviewer --name "$(pick_session_name)" \
    --output "$1/review/rendering" "rendering review" &
  run_agent --role bibtex_validator --name "$(pick_session_name)" \
    --output "$1/review/validation" "bibtex validation" &
}
run_phase5_review() { _arbiter_loop _phase5_reviewers "$1"; }

# Phases 4a, 4b: critical + plot validator + bibtex → arbiter.
_1bot_bib_reviewers() {
  run_agent --role critical_reviewer --name "$(pick_session_name)" \
    --output "$1/review/critical" "critical review" &
  run_agent --role plot_validator --name "$(pick_session_name)" \
    --output "$1/review/validation" "plot validation" &
  run_agent --role bibtex_validator --name "$(pick_session_name)" \
    --output "$1/review/validation" "bibtex validation" &
}
run_1bot_review_with_bibtex() { _arbiter_loop _1bot_bib_reviewers "$1"; }

# Phases 3, 4c: critical + plot validator, NO arbiter — gates on Category A/B.
run_1bot_review() {
  dir=$1; i=0
  while [ $i -lt $max_review_iterations ]; do
    i=$((i + 1))
    [ $i -gt 2 ] && echo "WARNING: 1-bot review iteration $i for $dir"
    run_agent --role critical_reviewer --name "$(pick_session_name)" \
      --output "$dir/review/critical" "critical review" &
    run_agent --role plot_validator --name "$(pick_session_name)" \
      --output "$dir/review/validation" "plot validation" &
    wait
    if ! review_has_category_a_or_b "$dir/review/critical" "$dir/review/validation"; then
      run_regression_check "$dir" || return 1
      check_upstream_feedback "$dir"; return 0
    fi
    if [ $i -ge 3 ]; then   # escalate after 3 non-converging iterations
      present_for_human_review "$dir"; wait_for_human_input; continue
    fi
    exec_name=$(pick_session_name)
    write_iteration_inputs_1bot "$dir" "$i" "$exec_name"
    run_agent --role fixer --name "$exec_name" \
      --output "$dir/outputs" "fix iteration v$((i+1))"
  done
  echo "ERROR: 1-bot review hit $max_review_iterations iterations for $dir"
  present_for_human_review "$dir"; wait_for_human_input; return 2
}

# --- Main pipeline ---
# *** EXAMPLE PATTERN *** channel_a/channel_b and calibration_1/calibration_2
# are placeholders — replace with analysis-specific names. Single-channel
# analyses drop the for-loops. Unified flow: 4a → 4b → human gate → 4c → 5.

run_agent --name "$(pick_session_name)" \
  --output "phase1_strategy/outputs" "execute phase 1"
run_2bot_review "phase1_strategy" || exit 1
git add phase*/ calibrations/ *.md pixi.toml && git commit -m "feat(phase1): strategy"

run_agent --name "$(pick_session_name)" \
  --output "phase2_exploration/outputs" "execute phase 2"
run_agent --role plot_validator --name "$(pick_session_name)" \
  --output "phase2_exploration/review/validation" "plot validation"
git add phase*/ calibrations/ *.md pixi.toml && git commit -m "feat(phase2): exploration"

# Phase 3 — per channel (parallel execution, sequential review)
for channel in channel_a channel_b; do
  run_agent --name "$(pick_session_name)" \
    --output "phase3_selection/channel_$channel/outputs" \
    "execute phase 3 ($channel)" &
done
wait
for channel in channel_a channel_b; do
  run_1bot_review "phase3_selection/channel_$channel" || exit 1
done
run_agent --name "$(pick_session_name)" \
  --output "phase3_selection/outputs" "consolidate channels"
git add phase*/ calibrations/ *.md pixi.toml && git commit -m "feat(phase3): selection"

# Shared calibrations (may run in parallel with phases 2-3, must finish
# before 4a — their scale factors/corrections are inputs to inference).
for cal in calibration_1 calibration_2; do
  run_agent --name "$(pick_session_name)" \
    --output "calibrations/$cal" "calibration: $cal" &
done
wait

# Phase 4a — expected results + AN v1 (agent gate). One executor role:
# systematics + expected results on Asimov, AN structure with expected-only
# results, compile AN to PDF for review.
run_agent --name "$(pick_session_name)" \
  --output "phase4_inference/4a_expected/outputs" \
  "execute phase 4a: statistics + write AN v1 (expected results) + compile PDF"
run_1bot_review_with_bibtex "phase4_inference/4a_expected" || { echo "Phase 4a review failed."; exit 1; }
git add phase*/ calibrations/ *.md pixi.toml && git commit -m "feat(phase4a): expected results"

# Phase 4b — 10% data validation + draft AN + human gate. One executor role:
# 10% stats, update AN from all phase artifacts, compile draft PDF.
run_agent --name "$(pick_session_name)" \
  --output "phase4_inference/4b_partial/outputs" \
  "execute phase 4b: 10% statistics + update AN + compile draft PDF"
run_1bot_review_with_bibtex "phase4_inference/4b_partial" || exit 1
present_for_human_review "phase4_inference/4b_partial"
wait_for_human_decision  # APPROVE / REQUEST CHANGES / HALT
git add phase*/ calibrations/ *.md pixi.toml && git commit -m "feat(phase4b): partial validation"

# Phase 4c — full data + AN update (one executor role).
run_agent --name "$(pick_session_name)" \
  --output "phase4_inference/4c_observed/outputs" \
  "execute phase 4c: full statistics + update AN with full results"
run_1bot_review "phase4_inference/4c_observed" || exit 1
git add phase*/ calibrations/ *.md pixi.toml && git commit -m "feat(phase4c): observed results"

# Phase 5 — documentation (one executor role + 2-bot review).
run_agent --name "$(pick_session_name)" \
  --output "phase5_documentation/outputs" \
  "execute phase 5: produce AN figures + write final AN + typeset final PDF"
run_phase5_review "phase5_documentation" || exit 1
git add phase*/ calibrations/ *.md pixi.toml && git commit -m "feat(phase5): analysis note"
```

---

## Appendix: Phase Dependency Graph

Unified flow for both measurements and searches:

```
                         Experiment Corpus (RAG)
                                │ queried throughout
                                ▼
Physics Prompt ──────► Phase 1: Strategy ◄──────────────────────┐
                                ▼                               │
                       Phase 2: Exploration ◄───────────────┐   │
                     ┌──────────┼──────────┐                │   │
                     ▼     (per channel)   ▼                │   │
                  Phase 3a            Phase 3b ...           │   │
                  Channel A           Channel B        phase regression
                     └──────────┬──────────┘          (if fundamental
                                ▼                       issue found)
                       Phase 4a: Expected Results           │   │
                        ★ AGENT GATE ★ (1-bot+bib) ──────────┘   │
                       Phase 4b: 10% Data Validation            │
                        ★ 1-BOT+BIB REVIEW ★ ──────────────────┘
                        ★ HUMAN GATE ★ (draft note + 10% results → human)
                       Phase 4c: Full Data
                                ▼
                       Phase 5: Documentation
```

Both analysis types share this structure. For searches, Phase 4b is a partial
unblinding (10% of signal-region data); for measurements, it is a consistency
validation (10% of data vs expected). The human gate sits between 4b and 4c in
both cases. See §3 (Phase 4) for details.

---

## Appendix: Tool Heuristics (Agent-Maintained)

A living document. On first encounter with a tool, the agent queries the tool's
current documentation, extracts best practices and pitfalls, and records a
concise summary here. Subsequent agents consult this appendix first and only
re-query upstream docs if something is out of date or missing.

**Purpose:** avoid repeated full-doc lookups — each entry captures what the
agent learned so the next session starts with working knowledge. Analogous to
the blessed-snippets library but for tool-level idioms and gotchas.

**Format:** one subsection per tool, including: version tested against; key
idioms / recommended patterns; common pitfalls and how to avoid them;
performance notes if relevant.

**Maintenance rules:**

- The agent **must** check this appendix before querying external docs for any
  tool listed in §7.1 (`01-core.md`).
- If an entry exists and is sufficient, use it — do not re-query.
- If an entry is missing or incomplete, query the current docs and use the
  result. **Do not modify this file during analysis execution** — it lives in
  the spec directory. Note the missing/outdated entry in the experiment log;
  updates happen after the analysis completes, following the same process as
  `conventions/` updates.
- Keep entries concise — heuristics and gotchas, not full API references.

### mplhep

**Version:** 1.1.0+

**Key idioms:**

- `mplhep.style.use("CMS")` sets the default style sheet (fonts, tick sizes,
  aesthetics). It does NOT add experiment labels.
- Add experiment labels via the generic `mplhep.label.exp_label(...)`, which
  works for any experiment. See `04-output.md` (plotting) for the full template
  and parameter reference (`exp`, `text`, `llabel`, `rlabel`, `data`, `com`,
  `lumi`, etc.).
- **Do NOT** hack around unwanted label text by patching rcParams or removing
  text objects after the fact. `exp_label` exposes all controls as kwargs —
  read `help(mplhep.label.exp_label)`.
- For ALEPH analyses using CMS style:
  `mplhep.label.exp_label(exp='ALEPH', data=True, rlabel=r'$\sqrt{s} = 91.2$ GeV')`
  gives clean labeling without CMS branding.

**Common pitfalls:**

- Using experiment-specific functions (e.g., `mplhep.cms.label`) instead of the
  generic `mplhep.label.exp_label` — the experiment-specific ones add unwanted
  branding.
- Forgetting `rlabel=''` and getting a default "(13 TeV)" CMS watermark. Always
  set `rlabel` explicitly.
- With `data=False`, mplhep auto-adds "Simulation" text; adding `llabel` on top
  stacks. Either use `data=False` alone, or `data=True, llabel="MC Simulation"`
  to fully control the left side.

**Performance:** no concerns — plotting is never the bottleneck.

---

## Appendix: Integration (Claude Code Adapter)

The methodology spec is portable; this section is the Claude Code adapter.

**Session logs.** Every agent session produces a `session.log`. Not read by
other agents (artifacts are the interface), but serves as an audit trail for
debugging and reproducibility.

**RAG integration.** The SciTreeRAG corpus (slopcorpus) is available to all
sessions via MCP. The server lazy-loads its index on first tool call;
subsequent calls within a session are fast. See `.mcp.json` for the server
definition. Expectations:

- All sessions have the MCP tools available — no per-session setup.
- Agents query as needed and cite sources in artifacts (paper ID + section).
- Failed retrievals are logged in `retrieval_log.md` per phase (see §2.2,
  `01-core.md`).
- Prefer `search_lep_corpus` with `mode="hybrid"` (default) for most queries;
  use `compare_measurements` to cross-check ALEPH vs DELPHI on the same
  observable; use `get_paper` to drill into a specific reference.

**Team structure.** The **lead agent** is the orchestrator — it spawns
teammates, manages dependencies, handles gates, and does no analysis work.
Per phase (2-bot example): Lead → {Executor, Physics Rev, Arbiter}. For 1-bot
phases the lead spawns only Executor + Critical Rev; for self-review phases,
only Executor.

**Isolation guarantees:**

- Each teammate has its own context window.
- Communication via shared files only.
- Parallel reviewers (e.g., physics reviewer and plot validator) cannot see
  each other's work.
- The experiment log is the only shared mutable state within a phase.

**Configuration:**

```json
{
  "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" },
  "mcpServers": {
    "lep-corpus": {
      "type": "stdio",
      "command": "pixi",
      "args": ["run", "--manifest-path", "/path/to/slopcorpus/pixi.toml",
               "python", "mcp_servers/rag_server.py"],
      "cwd": "/path/to/slopcorpus",
      "env": { "RAG_MODEL": "small" }
    }
  }
}
```

The `lep-corpus` MCP server exposes four tools:

| Tool | Purpose |
|------|---------|
| `search_lep_corpus(query, top_k, experiment, mode)` | Hybrid (dense + BM25) retrieval over ~2,400 ALEPH/DELPHI papers; ranked passages with metadata |
| `get_paper(paper_id)` | Look up a specific paper by arXiv, INSPIRE, or CDS ID |
| `list_corpus_papers(experiment, category, limit)` | Browse corpus with optional experiment/category filters |
| `compare_measurements(topic, top_k_per_experiment)` | Side-by-side ALEPH vs DELPHI retrieval for cross-checking |

All retrieved passages include source paper ID and similarity score — cite
these in artifacts.

**Adapting to other agent systems.** Requirements: isolated agent sessions with
file read/write and code execution; RAG corpus accessible as a tool; parallel
execution support; a mechanism to pause for human review; git integration.

---

## Appendix: Agent Prompt Templates

Full agent definitions — role, inputs/outputs, methodology references, and
prompt templates — live in `../agents/*.md`. See `../agents/README.md` for the
index and phase activation matrix. This is a summary of roles and context
assembly.

Context assembly follows §3a.4 (`02-phases.md`): three layers — bird's-eye
framing, relevant methodology sections, upstream artifacts. The phase CLAUDE.md
files (from `../templates/`) are what agents read at runtime; the agent
definitions specify how the *orchestrator* launches agents that read those files.

**Executor agents.** The executor is a single role that performs the
statistical analysis, writes the analysis-note prose, AND typesets it
(markdown → `.tex` → compiled PDF). At AN-producing phases (4a, 4b, 4c, 5) it
reads the phase artifacts and conventions, drafts the AN with full detail, and
runs the pandoc → `postprocess_tex.py` → tectonic chain (figure composition,
longtable conversion, compile→read→fix loop) to produce the PDF. PDF
compilation is mandatory before review at 4a, 4b, and Phase 5; the review panel
reads the compiled PDF. See §3 (Phase 5 typesetting) in `02-phases.md`.

| Role | Definition | Context | Writes |
|------|-----------|---------|--------|
| Executor | `agents/executor.md` | Full methodology + RAG | `outputs/` artifacts, `../src/` code, `outputs/figures/`, `outputs/ANALYSIS_NOTE_{phase}_v{N}.{md,tex,pdf}`, `logs/` |
| Fixer | `agents/fixer.md` | Arbiter verdict or regression ticket + existing code | Updated artifact + code, `logs/` |

**Reviewer agents:**

| Role | Definition | Context | Writes |
|------|-----------|---------|--------|
| Physics reviewer | `agents/physics_reviewer.md` | Physics prompt + artifact only | `review/physics/`, `logs/` |
| Critical reviewer | `agents/critical_reviewer.md` | Full methodology + RAG | `review/critical/`, `logs/` |
| Plot validator | `agents/plot_validator.md` | Plotting scripts + histogram data | `review/validation/`, `logs/` |
| BibTeX validator | `agents/bibtex_validator.md` | references.bib + web access | `review/validation/`, `logs/` |
| Rendering reviewer | `agents/rendering_reviewer.md` | Compiled PDF only | `review/rendering/`, `logs/` |

**Adjudication and specialist agents:**

| Role | Definition | Context | Writes |
|------|-----------|---------|--------|
| Arbiter | `agents/arbiter.md` | All reviews + artifact + conventions | `review/arbiter/`, `logs/` |
| Investigator | `agents/investigator.md` | Review output + origin phase | `REGRESSION_TICKET.md`, `logs/` |

**Execution + review by phase** (executor is a single role doing stats, AN
writing, and typesetting in one pass):

| Phase | Executor task | Review tier | Parallel agents | Then |
|-------|---------------|-------------|-----------------|------|
| 1 | executor | 2-bot | physics (+ plot validator at figure phases) | arbiter |
| 2 | executor | Self | executor self-check + plot validator | — |
| 3 | executor | 1-bot | critical + plot validator | (no arbiter) |
| 4a | executor (stats + AN v1 + compile PDF) | 1-bot+bib | critical + plot validator + bibtex | arbiter |
| 4b | executor (10% stats + update AN + compile PDF) | 1-bot+bib | critical + plot validator + bibtex | arbiter |
| 4c | executor (full stats + update AN) | 1-bot | critical + plot validator | (no arbiter) |
| 5 | executor (figures + final AN + final PDF) | 2-bot | rendering + bibtex + plot validator | arbiter |

---

## Appendix: Sessions and Directory Layout

### Session isolation

Every agent invocation — execution, review, arbitration — is a **separate,
isolated session** with explicitly defined inputs and outputs. No shared
conversation history, no shared memory, no implicit state. Each session reads
files, writes files, exits. The files are the interface.

```
┌─────────────┐     ┌──────────┐     ┌──────────────┐
│   inputs/   │────►│  agent   │────►│   outputs/   │
│ (read-only) │     │ session  │     │  (new files) │
└─────────────┘     └──────────┘     └──────────────┘
                         │
                         ▼   logs/{role}_{name}_{timestamp}.md
                         (full transcript via cc /export)
```

**Exception: the experiment log.** Each phase has an `experiment_log.md` that
persists across executor sessions within that phase. Every executor reads the
existing log and appends to it — the only mutable shared state within a phase.
It prevents agents from re-trying failed approaches and gives humans visibility.

**Concurrency note.** Parallel tracks (per-channel Phase 3, concurrent
calibrations) each have their own experiment log in their own directory. The
log is shared only across *sequential* sessions within a single track — never
across parallel agents. Sub-agents within one session that append to the same
log must do so sequentially.

### Agent session identity

Every session is assigned a **session name** — a random human first name from a
pool of 88 (from `orchestrator/names.py`), never reused within a run. Purposes:

1. **Traceability.** Every file produced includes the session name and
   timestamp, so agents and humans can trace who produced what and when.
2. **No clobbering.** Iteration produces new files rather than overwriting.
   Agents always read the most recent artifact (by timestamp); humans can
   compare versions.

Name pool: Ada, Agnes, Albert, Alfred, Amara, Andrzej, Anselm, Basil, Boris,
Brigitte, Casimir, Celeste, Claude, Cosima, Dagmar, Dmitri, Dolores, Dorothea,
Edmund, Eloise, Emeric, Eric, Estelle, Fabian, Fabiola, Felix, Fiona, Florence,
Gerald, Gertrude, Greta, Gunnar, Hana, Hedwig, Hiroshi, Hugo, Ingrid, Isolde,
Ivan, Jasper, Joe, Johanna, Jules, Katya, Kenji, Klaus, Lena, Leopold, Ludmila,
Magnus, Marcel, Margaret, Mireille, Nadia, Nikolai, Nora, Odette, Oscar, Otto,
Pavel, Peter, Petra, Phil, Philippa, Quentin, Rainer, Renata, Rosa, Sabine,
Sally, Sam, Sigrid, Sven, Theo, Tomas, Tomoko, Ulrich, Ursula, Valentina, Vera,
Viktor, Wanda, Wolfgang, Xena, Yuki, Yvette, Zelda, Zoran.

**Handoff file naming:** `{ARTIFACT}_{session_name}_{YYYY-MM-DD}_{HH-MM}.md`
(e.g. `STRATEGY_fabiola_2026-03-13_14-30.md`,
`STRATEGY_ARBITER_albert_2026-03-13_15-30.md`, `inputs_dolores_..._15-45.md`).
The orchestrator tells each agent its name in the input prompt; the agent uses
it when naming outputs. Downstream agents find the current artifact by the most
recent file matching the pattern (e.g. `STRATEGY_*_*.md`), sorted by timestamp.
No overwrites on iteration — history is self-documenting.

### Agent session logs

Each phase has a `logs/` directory centralizing all session records. Every
agent maintains an **incremental session log** — written progressively as the
agent works, not just at session end. This is the crash-resilient lab notebook:
if an agent dies mid-task, the log up to that point survives.

**File naming:** `logs/{role}_{session_name}_{YYYY-MM-DD}_{HH-MM}.md`
(e.g. `logs/executor_sam_2026-03-13_14-30.md`).

**When to append** — at natural milestones, not every thought: session start
(role, task, key inputs); plan produced (summary, not full plan); code written
(what script, key choices); test/validation run (what, result, interpretation);
figure generated (which, what it shows, issues); decision point (alternatives,
choice, why); error/retry; artifact produced; session end (accomplished,
remaining, open issues).

**Format.** Each entry is a timestamped markdown block, 2-5 sentences — a lab
notebook, not a transcript.

```markdown
### 14:48 — Preselection implemented
Wrote preselection.py: 4 cuts (ncharged>=5, thrust<0.9, Evis>0.5*sqrt(s),
|cos(theta)|<0.9). Cutflow: 847k→612k→589k→571k→498k. All cuts motivated by
EXPLORATION.md distributions.
```

**Supplement: `/export`.** At session end agents should still run `cc /export`
to save the full transcript alongside the session log. The transcript is the
complete record; the session log is the crash-safe summary. All session logs
for a phase live in one auditable location (`logs/`).

### Directory layout

One representative phase directory is shown in full; other phases follow the
same pattern (their `review/` subdirectory matches the phase's review tier, and
Phase 2 has no `review/` — self-review only). AN-producing phases (4a, 4b, 4c,
5) additionally hold `INFERENCE_*.md` and `ANALYSIS_NOTE_*.{md,tex,pdf}` (4b
also `UNBLINDING_CHECKLIST.md`) in `outputs/`.

```
analysis_name/
  prompt.md
  regression_log.md              # Tracks any phase regressions (if triggered)

  calibrations/                  # Shared sub-analyses (may grow to near-full
    calibration_1/               #   sub-analysis structure; e.g. btag,
      experiment_log.md          #   jet_corrections, trigger_eff
      retrieval_log.md
      CALIBRATION_1.md
      src/
      outputs/figures/
    calibration_2/ ...

  phase1_strategy/               # Representative phase directory
    experiment_log.md            # Lab notebook for this phase
    retrieval_log.md
    UPSTREAM_FEEDBACK.md         # Feedback from downstream phases (if any)
    REGRESSION_TICKET.md         # Regression investigation output (if triggered)
    src/                         # Phase-level codebase (can grow large)
    outputs/                     # All produced artifacts
      figures/
      inputs_fabiola_..._14-30.md            # Session-named inputs
      plan_fabiola_..._14-30.md
      STRATEGY_fabiola_..._14-30.md          # Session-named artifact
      # On iteration, new files appear alongside (no overwrites):
      # inputs_peter_..._16-00.md / STRATEGY_peter_..._16-00.md
    review/                      # Matches review tier (2-bot here: physics → arbiter)
      physics/
        STRATEGY_PHYSICS_REVIEW_andrzej_..._15-00.md
      validation/                # plot validator (+ bibtex validator at 4a/4b/5)
      arbiter/                   # present at 2-bot / 1-bot+bib / Phase 5 tiers
        STRATEGY_ARBITER_albert_..._15-30.md
    logs/                        # Incremental session logs + /export transcripts
      executor_fabiola_..._14-30.md
      physics_andrzej_..._15-00.md
      arbiter_albert_..._15-30.md

  phase2_exploration/            # Self-review only — no review/ directory
    experiment_log.md  retrieval_log.md  src/  outputs/figures/  logs/

  phase3_selection/              # Per-channel if multi-channel
    channel_a/                   # Replace with analysis-specific names
      experiment_log.md  retrieval_log.md
      sensitivity_log.md         # Tracks optimization attempts
      UPSTREAM_FEEDBACK.md  REGRESSION_TICKET.md
      src/  outputs/figures/  SELECTION_CHANNEL_A.md
      review/critical/           # 1-bot review per channel
      logs/
    channel_b/ ...               # same structure, SELECTION_CHANNEL_B.md
    SELECTION_COMBINED.md        # Consolidation artifact

  phase4_inference/
    4a_expected/                 # review/ = 1-bot+bib (critical + validation + arbiter)
    4b_partial/                  # adds UNBLINDING_CHECKLIST.md; 1-bot+bib then human
    4c_observed/                 # created after human approval; review/ = 1-bot

  phase5_documentation/          # review/ = 2-bot: validation + rendering → arbiter
    retrieval_log.md  UPSTREAM_FEEDBACK.md  REGRESSION_TICKET.md
    outputs/figures/  ANALYSIS_NOTE_5_v1.{md,tex,pdf}
    review/  logs/
```
