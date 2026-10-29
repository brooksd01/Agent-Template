---
name: start-session
description: Orient a fresh context — read the project's status docs and git state, then brief where we are and what's next. Run this first each session.
allowed-tools: Read Glob Grep Bash(git status:*) Bash(git log:*) Bash(git diff:*) Bash(git branch:*) Bash(svn status:*) Bash(svn log:*) Bash(svn diff:*) Bash(svn info:*)
---

You are starting a fresh working session on this project. Your job right now
is **orientation only** — get up to speed and brief the user. Do **not** start implementing
anything until they direct you.

> **If the project is still an uninitialized template** — `AGENTS.md` carries the 🌱 *uninitialized
> template* banner and STATUS says "Uninitialized" — don't produce a full brief. Tell the user the
> project hasn't been set up yet and recommend running **init-project** first. Then stop.

## Read, in this order

1. `AGENTS.md` (project root) — operating model and key facts. (In Claude Code it's already in
   context via `CLAUDE.md`; don't re-read it if so.)
2. `docs/1-documentation-index.md` — the documentation map. Note especially the "Start here" section and any
   docs marked ⚠️ Suspect or 🔴 Obsolete so you don't rely on stale material.
3. `docs/2-project-status.md` — the living current-state. Read it fully.
4. `docs/4-decision-log.md` — read the **last two dated sections** (most recent decisions) so you
   know what changed most recently. Don't re-read the whole file.
5. Working-copy state — `docs/6-working-model.md` records whether this project runs on git or
   svn. Git: `git status`, `git log --oneline -10`, and `git diff --stat` if there's uncommitted
   work. svn: `svn status`, `svn log -l 10`, and `svn diff --summarize` if there are local
   modifications. (If the project isn't under version control yet, note that and move on.)
   **Split layout** (`Layout: split` in `docs/6-working-model.md`): do this for the harness *and*
   for each code repo in the Code repositories table, by path — `git -C repos/<name> status`,
   `svn status repos/<name>` — noting each repo's current branch. A code repo missing from
   `repos/` means this machine isn't set up yet; say so.
6. Work orders (only if the project uses the coordinator/subagents model — check
   `docs/6-working-model.md` for the default, or the presence of `work-orders/` subdirectories
   with real orders). Orders live in per-branch subdirectories (`work-orders/<branch-slug>/`, or
   `work-orders/<repo>/<branch-slug>/` on split projects, keyed to each code repo's branch);
   scan the current branch's subdirectory first, then note any other branch's unfinished orders.
   For any order not yet `done`: `dispatched` or `in progress` usually means a subagent run was
   interrupted before reporting — the order needs re-dispatch (if its progress log has entries,
   the executor will resume from them); `ready to commit` means a result
   report is waiting on your review. You are booting as the **coordinator** — don't implement
   these; you review and dispatch.

If anything in STATUS references a doc you need detail on, open that specific doc — but don't
read the whole `docs/` tree speculatively. The INDEX tells you what's worth opening.

## Then brief the user

Produce a concise briefing with these parts:

1. **Where we are** — one short paragraph from STATUS + recent decisions.
2. **The single next action** — the one thing to do first, drawn from STATUS "Active now".
3. **Active / Up next / Blocked** — the near-term lists from STATUS, tightened to what matters.
4. **Working-tree state** — uncommitted changes or anything that looks mid-flight or drifted
   from what STATUS claims; flag discrepancies between STATUS and reality explicitly. On split
   projects, one line per repo (harness first), with its branch.
5. **Work orders in flight** (coordinator/subagents projects only) — any orders whose subagent
   run was interrupted (`dispatched` / `in progress` with no result report), and especially any
   `ready to commit` result reports awaiting your review. If a report is waiting, reviewing it is
   usually the single next action.
6. **Flags** — any open decisions awaiting the user, and any doc gaps STATUS calls out. If STATUS
   shows an **autonomy envelope** still active, or entries under **Parked (default applied)**, say
   so prominently: the parked defaults are provisional until the user answers them, and the
   envelope should be closed now that they're back.

**If STATUS clearly contradicts reality** — commits it doesn't know about, "Active now" items that
are visibly done on disk, a blocked item whose blocker is gone — don't just flag it: offer to
reconcile STATUS right now (a save-context-style sync) before work starts, so the session doesn't
run on known-stale state. Reconcile only with the user's go-ahead.

Keep it tight and scannable. End by asking what they want to work on — then stop and wait.
