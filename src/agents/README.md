# Agent Definitions

Self-contained definitions for every agent role in the analysis pipeline.
Each file is a complete, auditable specification: role, inputs, outputs,
methodology references, and prompt template. The orchestrator uses these
when spawning subagents.

Four roles. The note writer and typesetter were merged into the executor;
the review panel was collapsed to a single critical reviewer that also makes
the PASS / ITERATE call (no separate physics reviewer, plot validator, BibTeX
validator, rendering reviewer, or arbiter).

## Executor agents

| Agent | File | Role |
|-------|------|------|
| Executor | `executor.md` | Phase execution — plan, code, figures, artifacts, AND analysis-note writing + typesetting (PDF) |
| Fixer | `fixer.md` | Targeted fixes for review findings and regression tickets (replaces the executor during ITERATE cycles) |

## Reviewer agents

| Agent | File | Role |
|-------|------|------|
| Critical reviewer | `critical_reviewer.md` | The single reviewer at every gate — finds correctness/completeness flaws (including mechanical figure/citation lint) and makes the PASS / ITERATE call |

## Investigation agents

| Agent | File | Role |
|-------|------|------|
| Investigator | `investigator.md` | Regression investigation and scoped fix tickets |

## Phase activation

### Execution pipeline

The executor runs the whole producing step for each phase — including, from
Phase 4a on, writing the analysis note and typesetting the PDF in the same
step. The fixer replaces the executor during ITERATE cycles at any phase.

| Phase | Producing step |
|-------|----------------|
| Ph1 | executor (strategy) |
| Ph2 | executor (explore) |
| Ph3 | executor (selection) |
| Ph4a | executor (stats + AN v1 + compile PDF) |
| Ph4b | executor (10% stats + update AN + recompile PDF) |
| Ph4c | executor (full stats + update AN) |
| Ph5 | executor (figures + final AN + final PDF) |

### Review panel by phase

"x" = the critical reviewer is the gate at that phase. Phase 2 is
executor self-review (no independent reviewer).

| Agent | Ph1 | Ph2 | Ph3 | Ph4a | Ph4b | Ph4c | Ph5 |
|-------|-----|-----|-----|------|------|------|-----|
| Critical reviewer | x | | x | x | x | x | x |
| Self-review (executor) | | x | | | | | |

Mechanical figure lint and citation/render checks that earlier versions
split into separate validators are now part of the executor's self-lint and
PDF compilation. The critical reviewer covers correctness, completeness, and
the remaining figure/citation checks at every gate.

## Review panel composition

One critical reviewer per gate (Phase 2 is executor self-review). The
reviewer makes the PASS / ITERATE call directly — there is no separate
arbiter. Only Category A findings (genuine correctness errors) block
advancement; B/C are advisory. On ITERATE, the fixer addresses the
Category A findings and the orchestrator re-reviews; escalate to the human
only if stuck.

## Context assembly

Context assembly follows §3a.4 (three layers: bird's-eye framing, relevant
methodology sections, upstream artifacts). The phase CLAUDE.md files are
what agents read at runtime; these definitions specify how the
*orchestrator* launches the agents that read those files.
