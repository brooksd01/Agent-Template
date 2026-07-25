# [Project Name] — Project Orientation

> 🌱 **UNINITIALIZED TEMPLATE.** This project was created from the agent project template and
> hasn't been set up yet. Run the **init-project** workflow (`/init-project` in Claude Code, or
> follow `.claude/skills/init-project/SKILL.md` in any other agent) — it interviews you about the
> project, then fills in this file, the README, and the `docs/` baseline so every future session
> starts oriented. Delete this banner (and this whole note) once initialized.

This file is the canonical orientation for **any** coding agent working in this repo — it follows
the [AGENTS.md](https://agents.md) convention read natively by Codex, Cursor, Copilot, Gemini CLI,
and others. (Claude Code loads it via the thin `CLAUDE.md` shell.) It is the entry point for a
fresh context. Keep it short; the detail lives in the documents it points at.

## What this project is

_[One or two paragraphs: what we're building, for whom, and why it exists — the single most
important orientation a fresh context needs. Filled in by init-project.]_

## How we work

Agent sessions start with no memory of previous sessions, and there is no semantic search over
project knowledge. A small, deliberately-maintained set of documents *is* the project's memory —
committed to the repo, shared by the whole team, and readable by any tool:

- **`docs/1-documentation-index.md` is the map.** It lists every document, its purpose, and its
  status (verified / suspect / obsolete / scaffold). Consult the INDEX first, then open the specific
  doc — don't guess at filenames.
- **`docs/2-project-status.md` is the working memory.** "Where we are / what's next / what's
  blocked." First thing to read, last thing to update.

### The session workflow

Four workflow skills maintain the documents. They live in `.claude/skills/<name>/SKILL.md` in the
open [Agent Skills](https://agentskills.io) format. In Claude Code they are slash commands
(`/start-session` etc.); **in any other agent, when the user asks for one of these by name, read
and follow the corresponding `SKILL.md`**:

- **init-project** — one-time setup for a freshly-cloned template: interviews you and fills in this
  file, the README, and the `docs/` baseline. Run it once, at the very start.
- **start-session** — orients a fresh context: reads this file, the INDEX, STATUS, recent
  decisions, and git state, then briefs you and waits. Run it first each session.
- **save-context** — lightweight mid-session checkpoint: freshens STATUS only. Safe to run
  repeatedly; does *not* touch the decision log, roadmap, or deferred-items.
- **end-session** — full close-out: reconciles STATUS, appends decisions, records deferrals,
  stamps the INDEX. Run at a genuine close-out or when low on context.

### Keeping the loop honest (standing instructions for the agent)

- **Capture as you go.** When a significant task completes, a decision lands, or a new problem
  surfaces mid-session, make a note of it in the moment (a short line is enough) so the next
  checkpoint or close-out captures it instead of reconstructing it from memory.
- **Log decisions as they're made, not at close-out.** When an architectural or implementation
  decision lands, append it to `docs/4-decision-log.md` *now*, in that file's append-only style —
  don't wait for **end-session**. A session reset (crash, clear, compaction) must not lose
  rationale. **end-session** then only *verifies* the log is complete rather than writing it from
  memory. (This is why the log is append-only: an entry added mid-session is never rewritten.)
- **Offer the close-out.** If context is running low, or the user signals wrap-up ("that's it for
  today", "let's stop here"), proactively offer to run **end-session** (or at least
  **save-context**) rather than letting the session end with stale docs.
- **Never let known-stale state ride.** If you notice STATUS contradicting reality (git history,
  files on disk), say so and offer to reconcile immediately.

### Asking the developer questions

- **Default to text, not pop-ups.** When you need a decision or clarification, ask in the body of
  your reply — as numbered questions when there is more than one, so each can be answered in turn.
  Do **not** reach for structured/pop-up question prompts (e.g. Claude Code's AskUserQuestion)
  unless the developer explicitly asks for that format; they hide the surrounding text the reader
  needs before answering.

## The document hierarchy

| File | Horizon | Role |
|---|---|---|
| `docs/2-project-status.md` | **Now** | Living current-state + near-term checklist + blockers. Churns constantly. |
| `docs/3-project-roadmap.md` | **Strategic** | Phase-by-phase plan and status. Changes when a phase moves. |
| `docs/5-deferred-items.md` | **Someday/maybe** | Consciously-punted items, each with a revisit *trigger*. |
| `docs/4-decision-log.md` | **Permanent** | Append-only, dated decisions *and their rationale*. Never rewrite; append. |
| `docs/6-working-model.md` | **Operating** | How sessions are structured (solo vs. coordinator/executor) and the hard git rule. |

`docs/1-documentation-index.md` sits above these as the catalogue of the full `docs/` set.

## Working model & the git rule

Sessions run in one of two modes, defined in `docs/6-working-model.md` (which records this
project's default):

- **Solo** — one session plans, implements, and reviews. The default for small work.
- **Coordinator / executor** — a premium model coordinates (plans, decides, reviews) while cheaper
  executor sessions and subagents implement, each driven by a written **work order**
  (`work-orders/NNN-slug.md`). This keeps premium capacity on thinking and the coordinator's context
  light. Dispatch with the **dispatch-work-order** skill; run an order with **execute-work-order**.

The mode is a default, not a lock — a solo project can still dispatch one heavy work order.

**Git rule (hard, both modes):** the agent **never commits without explicit, per-commit human
permission, and never pushes, rebases, or resets** — in any role, in any session. Its job is to
*remind* when work is review-ready; commit messages are human-written where possible, and any
agent-drafted message is a suggestion to edit, not an auto-applied commit. An executor ends its work
at "ready to commit," never at "committed." (One narrow, opt-in exception — local
autonomous-checkpoint commits for throwaway/exploratory work — is documented in
`docs/6-working-model.md`.)

## Facts worth knowing before you touch anything

_[Project-specific gotchas a fresh context would otherwise learn the hard way — non-obvious
constraints, read-only or off-limits areas, external dependencies, environment quirks. Filled in by
init-project; grows over time. Starts empty.]_
