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
- **In an unattended run, park — don't block.** Only when the human has explicitly granted an
  autonomy envelope (see "Unattended runs" in `docs/6-working-model.md`): a question short of a
  real blocker gets the most conservative reasonable default, and is recorded under **Parked** in
  STATUS for the human to answer later. Outside an envelope, ask as usual.

## The document hierarchy

| File | Horizon | Role |
|---|---|---|
| `docs/2-project-status.md` | **Now** | Living current-state + near-term checklist + blockers. Churns constantly. |
| `docs/3-project-roadmap.md` | **Strategic** | Phase-by-phase plan and status. Changes when a phase moves. |
| `docs/5-deferred-items.md` | **Someday/maybe** | Consciously-punted items, each with a revisit *trigger*. |
| `docs/4-decision-log.md` | **Permanent** | Append-only, dated decisions *and their rationale*. Never rewrite; append. |
| `docs/6-working-model.md` | **Operating** | How sessions are structured (solo vs. coordinator/subagents), the project's layout, VCS, parallel-dispatch, and model-tier defaults, the split layout, unattended runs, and the hard commit rule. |

`docs/1-documentation-index.md` sits above these as the catalogue of the full `docs/` set.

## Working model & the git rule

Sessions run in one of two modes, defined in `docs/6-working-model.md` (which records this
project's defaults — mode, version control, parallel dispatch, model tiers):

- **Solo** — one session plans, implements, and reviews. The default for small work.
- **Coordinator / subagents** — one long-lived session coordinates (plans, decides, reviews) while
  **executor subagents** implement, each driven by a written **work order**
  (`work-orders/<branch-slug>/NNN-slug.md` — numbered per branch so branches never collide; on a
  split-layout project, `work-orders/<repo>/<branch-slug>/NNN-slug.md`). This
  keeps the coordinator's context light, with mechanical work optionally on a cheaper model tier.
  Model choice per role lives in one table in `docs/6-working-model.md`. Dispatch with the
  **dispatch-work-order** skill (which spawns the executor); the executor's contract is the
  **execute-work-order** skill — in tools without subagents, a separate human-opened session
  follows that same skill as the fallback vehicle.

The mode is a default, not a lock — a solo project can still dispatch one heavy work order.

**Version-control rule (hard, both modes, git or svn):** the agent **never commits without
explicit, per-commit human permission, and never pushes, rebases, or resets** — in any role, in any
session or subagent. On **svn** projects the bar is higher: `svn commit` publishes straight to the
server, so agent commits are denied outright — the human always runs the commit. The agent's job is
to *remind* when work is review-ready; commit messages are human-written where possible, and any
agent-drafted message is a suggestion to edit, not an auto-applied commit. An executor ends its work
at "ready to commit," never at "committed." (One narrow, opt-in exception — local
autonomous-checkpoint commits for throwaway/exploratory work, **git projects only** — is documented
in `docs/6-working-model.md`.) On a split-layout project the rule covers **every** repo — the
harness and each code repo, each under its own VCS.

## Repository layout

_[Filled in by init-project. Single: "This repo holds both the code and the project docs." Split:
one line per code repo — name, path under `repos/`, and what it is. The full table (VCS, remote,
default branch) lives in `docs/6-working-model.md`.]_

**Split-layout rules** _(only when `docs/6-working-model.md` records `Layout: split`;
init-project removes this block on single-repo projects)_:

- **Launch from the harness root, always.** Settings, deny rules, and hooks load only from the
  directory the agent is launched in. Opened inside a code repo, none of the guardrails apply.
- **Run VCS commands against code repos by path** — `git -C repos/<name> status`,
  `svn status repos/<name>` — rather than `cd repos/<name> && …`.
- **Docs live where their readers are.** Anything needed to build, run, operate, or maintain the
  code — setup, architecture, runbooks, schema and migration notes — lives in the code repo, so a
  developer without this harness still has it. The process lives here: STATUS, roadmap, work
  orders, deferred items, the decision log. When a decision changes the architecture, update the
  code repo's architecture doc as well as logging it here. The INDEX catalogues both, with paths
  into `repos/`.
- **Whose instructions win.** A code repo may carry its own `CLAUDE.md` or `AGENTS.md` (some
  agents load these automatically once they touch files there). On **process** — commits, the
  session workflow, standing rules — this harness wins. On **code conventions** — style, patterns,
  build and test commands — the code repo's own files win. A genuine conflict between the two is
  escalated to the human, never settled silently.
- **Each code repo has an `AI-HARNESS.md` at its root** pointing back here, for reference and
  history. Contributors don't need this harness to work on the code.

## Facts worth knowing before you touch anything

_[Project-specific gotchas a fresh context would otherwise learn the hard way — non-obvious
constraints, read-only or off-limits areas, data access and sensitivity (what the agent may see and
where it may write), external dependencies, environment quirks. Filled in by init-project; grows
over time. Starts empty.]_

## Standing rules

Rules that bind **every** session and every work order, whether or not a given order repeats them.
Executors read this section at boot. Each rule is one short, checkable line — the thing to do or
never do, plus a pointer to the decision-log entry that explains why.

How rules get here: when a lesson applies beyond the task that taught it — most often a bug fixed
at the shared level because it could recur across screens, endpoints, or orders — the agent
proposes a rule guarding against the whole class. The human approves it, and it's logged in
`docs/4-decision-log.md` when it lands. A lesson learned once is then enforced every time after,
instead of being rediscovered. Retire a rule the same way: proposed, approved, logged.

_[None yet.]_
