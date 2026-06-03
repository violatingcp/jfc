# Methodology Specification

Condensed methodology for the LLM-driven HEP analysis framework — six files
(consolidated from the original 21). Read the relevant one as needed; the
original section numbers (§1–§12) are preserved so cross-references stay valid.

| File | Covers | When |
|------|--------|------|
| `01-core.md` | Scope & principles (§1); inputs, incl. the numeric-constant citation policy (§2, §2.3); tools & paradigms (§7) | Project start; coding phases |
| `02-phases.md` | Analysis phases 1–5 incl. 4a/4b/4c (§3); orchestration & agent architecture (§3a); artifact format (§5) | Before each phase; writing artifacts |
| `03-review.md` | Review protocol — single-reviewer gate (§6.2), focus by phase (§6.4), reviewer scoping (§6.5.1), human gate / non-destructive regression (§6.6), phase regression (§6.7), validation-target / 3σ rule (§6.8); plus the review checklist | Spawning the reviewer; Phase 5 |
| `04-output.md` | Analysis-note specification (required sections, depth, formatting) and the plotting template/rules | Phase 4 (writing AN), figure-producing phases, Phase 5 |
| `05-practices.md` | Coding & version control (§11, §11.4); scope management / downscoping (§12); blinding / staged validation (§4); multi-channel analyses (§9) | Coding phases; Phase 4; hitting limitations |
| `06-appendix.md` | Automation pipeline, dependencies, heuristics, integration notes, agent prompt templates, session/directory layouts | As referenced |

## Reading guide

- **For a new analysis:** Read `02-phases.md` to understand what each phase does and produces.
- **For orchestration:** `02-phases.md` §3a for the architecture; `06-appendix.md` for prompt templates, automation pipeline, and session/directory layout.
- **For coding & output:** `01-core.md` (§7 tools) and `05-practices.md` for code style; `04-output.md` for the analysis-note spec and plotting rules.

**Scope reminders (already baked in):** the critical reviewer runs at Phases
1, 3, and 4a only and blocks on Category-A correctness errors (see
`03-review.md` §6.2); Phase 2 is self-review; 4b goes to the human gate; 4c
and 5 are covered by the orchestrator's regression checklist (no reviewer).
The executor writes *and* typesets the analysis note — there are no separate
note-writer or typesetter roles.
