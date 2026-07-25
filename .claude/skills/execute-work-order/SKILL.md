---
name: execute-work-order
description: Executor role — implement a single work order in a disposable session, then write a structured result report and stop at ready-to-commit. Boots minimal (AGENTS.md facts + the one work order only) to keep context tight. Takes the work-order number as its argument.
allowed-tools: Read Glob Grep Edit Write Bash
---

You are acting as the **executor** for one work order. Your context is disposable by design — do
the work, report it, and stop. Everything durable lives in the code and the result report, not in
this conversation. See `docs/6-working-model.md` for the working model; you do **not** need to read
the rest of the doc set.

## 1. Boot minimal

Read only:

1. `AGENTS.md` — but only the **"Facts worth knowing before you touch anything"** section (and the
   git rule). Skip the session-workflow and doc-hierarchy detail; that's coordinator concern.
2. The work order itself — `work-orders/NNN-slug.md` for the number you were given. If you weren't
   given a number, ask which one (or list the `dispatched` orders in `work-orders/` and confirm).

Do **not** read STATUS, the roadmap, the decision log, or unrelated docs. Keeping this context
tight is the whole point of the executor tier. If the work order references a specific doc as
needed context, read that one — nothing more.

## 2. Implement strictly from the order

- Do exactly what the order specifies. Meet the acceptance criteria; respect the constraints.
- **Implement what's specified; surface what's ambiguous.** If you hit a real decision the order
  doesn't settle (an architectural fork, an unspecified trade-off), do **not** invent an answer —
  note it as an escalation and either stop or proceed on the most conservative reading, as the
  order directs.
- Run the build-test-fix loop as needed. If local processes hold output locks, build/test to a
  scratch artifacts path rather than fighting the lock.
- Mark **Status: in progress** on the work order while you work.

## 3. Write the result report

Append to the same `work-orders/NNN-slug.md` file, filling in the **Result report** half of the
template: outcome, what changed (pointers not diffs), build/test verdicts (point to a scratch log
path rather than pasting bulk output), deviations from the order, and open questions/escalations.
Terse and structured — the coordinator reviews from this, so pointers beat payloads.

Set **Status: ready to commit** (or `blocked` / `partial` if that's the truth).

## 4. Stop at ready-to-commit

**Do not commit.** End at "ready to commit" and say so. The git rule is hard: no commit without
explicit per-commit human permission, and never push/rebase/reset. If the work is review-ready, the
result report already says so — that's your reminder to the human, not a license to commit.

(The only exception is a project explicitly in autonomous-checkpoint mode — see the git-rule
exception in `docs/6-working-model.md` — and even then, only local commits, never push.)

Then stop. Don't pick up the next work order or update the five docs — that's the coordinator's
job. This session can be reset once the report is written.
