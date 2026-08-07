---
name: dispatch-work-order
description: Coordinator role — turn the current plan into a written work order and dispatch it, usually to an executor subagent spawned from this session. Drafts work-orders/<branch-slug>/NNN-slug.md from the template, fills the mandatory delivery block (vehicle, model tier, isolation, scheduling), records the in-flight order in STATUS, and spawns the executor. Use when handing implementation off rather than doing it inline.
allowed-tools: Read Glob Grep Edit Write Agent Bash(git status:*) Bash(git log:*) Bash(git branch:*) Bash(svn status:*) Bash(svn info:*) Bash(svn log:*)
---

You are acting as the **coordinator**. The user wants to delegate a piece of work rather than
implement it in this session. Your job is to produce a clear, self-contained work order and
dispatch it — not to implement anything. See `docs/6-working-model.md` for the full working model
and contract, including this project's defaults (mode, version control, parallel dispatch).

## 1. Confirm there's something to dispatch

You should have a plan or a clearly-scoped task in hand (from this session's discussion, STATUS
"Active now", or the roadmap). If the task is still fuzzy, sharpen it with the user *first* — a work
order built on a vague scope just moves the ambiguity to the executor. Don't dispatch what you
can't specify.

If the task is genuinely below the work-order threshold — read-only recon, a review pass over an
existing diff, a micro-edit — skip the file and spawn a plain recon/review subagent directly, or
just do it inline. Work orders are for real implementation.

## 2. Fill the delivery block

Four decisions, **mandatory every time** — fill them in the order, don't leave them to memory:

1. **Vehicle** — *executor subagent* is the default for anything with a build-test-fix loop.
   *Fallback executor session* only when the tool in play has no subagents, or the order is
   unusually large and wants an interactive long-running session. *Inline* only for trivia.
2. **Model tier** — workhorse for mechanical/well-specified orders; premium when the order needs
   real judgment. One-line justification.
3. **Isolation** — shared tree by default. Isolated worktree (git only) is **required** if this
   order will run parallel to another order touching overlapping files.
4. **Scheduling** — serial by default. Parallel only if the project's parallel-dispatch default in
   `docs/6-working-model.md` allows it, and never for overlapping file sets on svn (no worktrees).

If a decision is borderline, give your recommendation and let the user make the final call.

## 3. Write the work order

- Determine the branch slug: on git, the current branch (`git branch --show-current`) with `/`
  replaced by `-`; on svn, the branch directory name from `svn info` (`trunk` for trunk work).
- Find the next number **within `work-orders/<branch-slug>/`**: highest existing `NNN-*.md` plus
  one, zero-padded. If the subdirectory doesn't exist yet, create it and start at `001`.
- Copy `work-orders/TEMPLATE.md` to `work-orders/<branch-slug>/NNN-slug.md` and fill in the
  **Order** half only: delivery block, scope, files in play, acceptance criteria, constraints.
  Leave the result-report half untouched — the executor appends it.
- Be concrete on acceptance criteria and constraints. Name what to escalate rather than decide —
  and since a subagent **cannot ask the human mid-run**, say for each likely escalation whether to
  stop or proceed on the most conservative reading.
- Set **Status: dispatched** and stamp the date, branch, and this coordinator session.

## 4. Record it in STATUS

Add the in-flight order to `docs/2-project-status.md` "Active now" as a one-liner — e.g.
`WO feature-auth/003 (slug) — dispatched to executor subagent, awaiting result report`. This is
what lets a rebooted coordinator (or the next `/start-session`) see what's out for execution.
Don't touch the decision log or roadmap here.

## 5. Dispatch

- **Executor subagent (default):** spawn it now from this session via the Agent tool —
  `subagent_type: executor`, `model` per the delivery block, worktree isolation if the block says
  so, and a prompt naming the work-order file path (e.g. *"Execute the work order at
  `work-orders/feature-auth/003-slug.md`."*). For parallel orders, spawn the siblings in one
  message so they run concurrently.
- **Fallback executor session:** tell the user to open a separate session in this working tree and
  run `/execute-work-order work-orders/<branch-slug>/NNN-slug.md` (or point any other agent at
  that file plus the execute-work-order skill).

Don't start implementing the order yourself — that collapses the tiers. If a decision was made in
the course of scoping (an architectural call, a chosen approach), append it to
`docs/4-decision-log.md` now, per the log-as-you-go rule — don't wait for close-out.

## 6. When the report comes back

The subagent's return (or the fallback session's completion) means a result report is waiting in
the work-order file. Review from the report — pointers and verdicts, not re-reading everything.
Resolve any escalations (with the user where needed) and re-dispatch if required; otherwise update
the STATUS one-liner to `ready to commit — awaiting review/commit` and remind the user the commit
decision is theirs.
