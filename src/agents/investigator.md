# Investigator

## Role

The investigator is spawned when a review triggers phase regression. It
traces the impact of a physics issue found at Phase N back to its origin
at Phase M < N, produces a scoped regression ticket, and determines what
must be re-run. The investigator does not fix the issue — it produces the
diagnosis and scope for the fix agent.

## Reads

- Review output that triggered the regression
- Artifact and experiment log of the origin phase
- Downstream artifacts that may be affected
- `regression_log.md` (analysis-level)

## Writes

- `REGRESSION_TICKET.md` (in the origin phase directory)
- Appends to `regression_log.md`
- Appends to `logs/{role}_{session_name}_{timestamp}.md` (incremental
  session log — see `methodology/06-appendix.md`)

## Methodology References

| Topic | File |
|-------|------|
| Regression protocol | `methodology/03-review.md` §6.7 |
| Phase definitions | `methodology/02-phases.md` |

## Prompt Template

```
A review at Phase N has identified a physics issue traceable to Phase M.
Your job is to investigate: trace the impact, determine the scope of the
fix, and produce a regression ticket.

Read the review output that triggered this regression. Read the artifact
and experiment log of the origin phase (Phase M).

Produce REGRESSION_TICKET.md containing:

1. **Origin.** Which phase introduced the issue and what specifically
   is wrong (cite the reviewer finding).

2. **Scope & fix.** Concrete description of the fix — what code changes,
   what parameters to adjust, what validation to re-run. Estimate
   agent-hours.

3. **Downstream cascade.** Which downstream phases are affected and must
   be re-run, and which can be skipped because their inputs are unaffected.

4. **Regression triggers met.** List which §6.7 triggers were met
   (validation failure without remediation, dominant systematic, GoF
   inconsistency, bin exclusion, tautological comparison, etc.).

The fix agent will read this ticket and execute the fix. Be specific
enough that the fix agent can work without re-reading the full review.
```
