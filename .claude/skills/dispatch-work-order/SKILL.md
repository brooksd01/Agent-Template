---
name: dispatch-work-order
description: Coordinator role — turn the current plan into a written work order for an executor session or subagent. Drafts work-orders/NNN-slug.md from the template, sets the mandatory delivery-vehicle field, and records the in-flight order in STATUS. Use when handing implementation off rather than doing it inline.
allowed-tools: Read Glob Grep Edit Write Bash(git status:*) Bash(git log:*)
---

You are acting as the **coordinator**. The user wants to delegate a piece of work rather than
implement it in this session. Your job is to produce a clear, self-contained work order — not to
implement anything. See `docs/6-working-model.md` for the full working model and contract.

## 1. Confirm there's something to dispatch

You should have a plan or a clearly-scoped task in hand (from this session's discussion, STATUS
"Active now", or the roadmap). If the task is still fuzzy, sharpen it with the user *first* — a work
order built on a vague scope just moves the ambiguity to the executor. Don't dispatch what you
can't specify.

## 2. Pick the delivery vehicle

The vehicle field is **mandatory** — fill it every time, don't leave it to memory:

- **Executor session** — anything with a build-test-fix loop, multi-file implementation, or
  meaningful iteration. This is the default for real code work.
- **Subagent** — read-only recon, a review pass over an existing diff, or a micro-edit too small to
  justify a separate session. Spawn these from *this* session via the Agent tool; they don't need a
  committed work-order file unless you want the record.

State the vehicle and a one-line justification. If it's borderline, give your recommendation and
let the user make the final call.

## 3. Write the work order

- Find the next number: look in `work-orders/` for the highest `NNN-*.md` and add one (zero-padded,
  e.g. `003`). If the directory only has `TEMPLATE.md`, start at `001`.
- Copy `work-orders/TEMPLATE.md` to `work-orders/NNN-slug.md` and fill in the **Order** half only:
  vehicle + justification, scope, files in play, acceptance criteria, constraints. Leave the result
  report half untouched — the executor appends it.
- Be concrete on acceptance criteria and constraints. Name what to escalate rather than decide.
- Set **Status: dispatched** and stamp the date and this coordinator session.

## 4. Record it in STATUS

Add the in-flight order to `docs/2-project-status.md` "Active now" as a one-liner — e.g.
`WO-003 (slug) — dispatched to executor, awaiting result report`. This is what lets a rebooted
coordinator (or the next `/start-session`) see what's out for execution. Don't touch the decision
log or roadmap here.

## 5. Hand off

Tell the user the work order is ready and how to run it:

- **Executor session:** open a separate session in this same working tree and run
  `/execute-work-order NNN` (in Claude Code) or point the agent at `work-orders/NNN-slug.md` and
  the execute-work-order skill.
- **Subagent:** offer to spawn it now from this session.

Then stop. Don't start implementing the order yourself — that collapses the two tiers. If a decision
was made in the course of scoping (an architectural call, a chosen approach), append it to
`docs/4-decision-log.md` now, per the log-as-you-go rule — don't wait for close-out.
