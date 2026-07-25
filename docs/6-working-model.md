# Working Model — Coordinator / Executor

**Purpose:** How agent sessions are structured on this project. Two operating modes: **solo**
(one session does everything) and **coordinator/executor** (a premium model coordinates; cheaper
models execute). This doc defines the coordinator/executor model and the rules that hold whichever
mode is in play. `init-project` records the project's default mode below.

**Project default mode:** _[set by init-project — `solo` or `coordinator/executor`]_

> The mode is a **default, not a lock.** A solo-default project can still dispatch a work order for
> one heavy task; a coordinator/executor project can still do small things inline. The default just
> tells a freshly-booted session what to expect and reminds it the option exists.

---

## Why this model

Two goals: spend premium-model capacity only on planning, decisions, and review; and keep the
coordinator's context light and long-lived by pushing all detail churn into disposable contexts.
The five-doc system is what makes the coordinator *cheaply rebootable* — long-lived is the
optimization, cheaply-reborn is the fallback, and the docs are the fallback's fuel.

## Roles

**Coordinator** (premium model; one long-lived session — the normal `/start-session` session)
- Owns planning, task decomposition, architectural decisions, and result review.
- Writes work orders; reads result reports. Delegates bulk review rather than reading diffs and
  logs directly — **but** this is a context-budget rule, not an absolute: when a diff is smaller
  than the review verdict would be, just read it. Delegate when reading it would cost more context
  than the verdict returns.
- Never edits code beyond trivial micro-fixes (a doc stamp, a one-liner).
- Conversation with the human is the only unavoidable context weight; everything else arrives as
  summaries and pointers.

**Executor** (workhorse model; separate session, same working tree — booted with
`/execute-work-order`)
- Does all real implementation: anything with a build-test-fix cycle.
- Operates strictly from the work order; escalates decisions back rather than making them —
  *implement what's specified; surface what's ambiguous.*
- Boots minimal on purpose: reads `AGENTS.md`'s facts and the one work order, **not** the full doc
  set. Its context stays as tight as the coordinator's.
- Writes a structured result report per work order, then stops. **Reset freely** — its context is
  disposable by design; anything durable lives in the docs and the result report.

**Subagents** (spawned from the coordinator)
- Read-only recon ("how does X work", codebase sweeps) via cheap search agents.
- Review passes over an executor's diff — the coordinator reads only the findings.
- Micro-tasks too small to justify an executor handoff.

## The work-order contract

Delegation always goes through a written work order at `work-orders/NNN-slug.md`, created from
`work-orders/TEMPLATE.md`. It carries scope, files in play, acceptance criteria, constraints, and a
mandatory **delivery-vehicle field** — *subagent or executor session, with a one-line
justification*. The vehicle decision is structural, not remembered: every delegation fills in the
blank. Default to the **executor session** for anything with a build-test-fix loop; reserve
**subagents** for recon, review, and micro-edits. The human makes the final call when the
recommendation is borderline.

The executor answers with a **result report** appended to the same file: terse, structured,
pointers over payloads — what changed, test/build results, deviations from the order, and open
questions. The coordinator reviews from this report.

Work orders and their result reports are **committed to the repo.** They are project
documentation — the record of what was attempted, what worked, and what didn't in the build
process — and they are the paper trail the terse-commit git rule below leans on.

## Context discipline

- Reports return **pointers + verdicts**, not inline dumps.
- Decisions land in `docs/4-decision-log.md` **as they're made**, not at session end. A coordinator
  reset (crash, clear, compaction) must not lose rationale. See the log-as-you-go standing
  instruction in `AGENTS.md`.
- The project's docs must always be sufficient to reboot a fresh coordinator cheaply. If they
  aren't, that's the bug — fix the docs, not the model.

## Git rule (hard)

**The agent never commits without explicit, per-commit human permission — in any role, in any
session — and never pushes, rebases, or resets.** Its job is to *remind*: when work is
review-ready, the result report (or coordinator summary) says so and suggests committing. Commit
messages are **human-written where possible**; an agent-drafted message is a suggestion to edit,
not an auto-applied message. Rationale: for professional work, commit authorship and history stay
in human hands; the docs, code, and work orders carry any detail a terser human message omits.

Corollaries:
- The executor ends its work orders at "ready to commit," never at "committed."
- One pair of human hands owns the working tree's git state; sessions never rebase, reset, or push.

**Enforcement.** `push`, `rebase`, and `reset` are denied in tracked `.claude/settings.json`
(Claude Code) — these never lift. `commit` is *not* denied; it stays a per-commit permission
prompt, and that prompt is the human approval the rule requires. In other tools, the written rule
above carries the weight — the deny rules are Claude Code-specific.

### Exception: autonomous-checkpoint mode (opt-in, per-project, local)

For **exploratory or throwaway work** where you want the agent to run a long multi-step process
largely unattended — and you want checkpoints to roll back to — you can allow autonomous commits by
adding `Bash(git commit:*)` to `.claude/settings.local.json` (gitignored, so it never ships with
the template or becomes a project default). Push/rebase/reset stay denied, so the checkpoints are
**purely local** — exactly what rollback wants. Use this only on projects unlikely to reach
production in their initial form; never as a default on professional work.

## Working-tree hygiene (two sessions, one directory)

- Executor builds/tests to a scratch artifacts path if local processes hold output locks.
- Coordinator review happens *after* the executor reports done and *before* the next work order
  dispatches to the same area — no concurrent edits to the same files from two sessions.
- Only the coordinator writes the five docs; the executor writes code and its result report. That
  single-writer rule is what keeps two sessions from clobbering shared docs.
- The human is the arbiter when coordinator review and the next work order would race.

## Failure modes this guards against

- **Premium-model burn on mechanical work** → executor tier.
- **Coordinator context bloat → compaction → plan drift** → churn quarantined in disposable
  contexts; docs as rebootable memory.
- **"Start a new thread" guidance skipped mid-flow** → the vehicle decision is a mandatory
  work-order field, not a disposition to remember.
- **Agent-authored commits in professional history** → hard git rule.
- **Two sessions clobbering one tree** → single git owner, single doc-writer, sequenced dispatch.
