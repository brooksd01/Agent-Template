# Working Model — Coordinator / Subagents

**Purpose:** How agent sessions are structured on this project. Two operating modes: **solo**
(one session does everything) and **coordinator/subagents** (a premium session coordinates;
subagents execute work orders). This doc defines the coordinator/subagent model and the rules that
hold whichever mode is in play. `init-project` records the project's defaults below.

**Project defaults** _(set by init-project)_:

- **Mode:** _[`solo` or `coordinator/subagents`]_
- **Version control:** _[`git` or `svn`]_
- **Parallel dispatch:** _[`serial` (default) or `parallel-allowed` — git projects only; svn is
  always serial for overlapping work. See "Working-tree hygiene."]_

> These are **defaults, not locks.** A solo-default project can still dispatch a work order for one
> heavy task; a coordinator project can still do small things inline; a serial project can run one
> deliberate parallel batch. The defaults just tell a freshly-booted session what to expect.

---

## Why this model

Two goals: spend premium-model capacity only on planning, decisions, and review; and keep the
coordinator's context light and long-lived by pushing all detail churn into disposable contexts.
The five-doc system is what makes the coordinator *cheaply rebootable* — long-lived is the
optimization, cheaply-reborn is the fallback, and the docs are the fallback's fuel.

## Roles

**Coordinator** (premium model; one long-lived session — the normal `/start-session` session)
- Owns planning, task decomposition, architectural decisions, and result review.
- Writes work orders; dispatches subagents against them; reads result reports. Delegates bulk
  review rather than reading diffs and logs directly — **but** this is a context-budget rule, not
  an absolute: when a diff is smaller than the review verdict would be, just read it.
- Never edits code beyond trivial micro-fixes (a doc stamp, a one-liner).
- Conversation with the human is the only unavoidable context weight; everything else arrives as
  summaries and pointers.

**Executor subagents** (the default delivery vehicle; spawned from the coordinator via the Agent
tool as the `executor` agent type)
- Do all real implementation: anything with a build-test-fix cycle.
- Operate strictly from the work order; the full behavioral contract lives in
  `.claude/skills/execute-work-order/SKILL.md` — the agent definition in
  `.claude/agents/executor.md` is a thin pointer to it (single source of truth, no drift).
- Boot minimal on purpose: the skill, `AGENTS.md`'s facts, and the one work order — **not** the
  full doc set.
- **Cannot ask the human questions mid-run.** A subagent that hits a real decision the order
  doesn't settle either proceeds on the most conservative reading or stops — and records the
  escalation in its result report either way. The coordinator resolves it (with the human where
  needed) and re-dispatches.
- Write a structured result report per work order, then stop at **ready to commit**. Their context
  evaporates when they return — anything durable lives in the code and the report.
- Model tier is a **per-dispatch decision**, not fixed: the coordinator passes `model` on the
  Agent call (workhorse tier for mechanical orders; same-tier for genuinely hard ones).

**Fallback: executor session** (a separate human-opened session, same working tree)
- The same contract, delivered by hand: open a session, run `/execute-work-order <path>` (or point
  any other agent at the same SKILL.md). Use it when the tool in play has no subagents
  (Codex/Cursor/Gemini/etc.), or for an unusually large order that wants an interactive,
  long-running session.
- Everything in this doc that applies to executor subagents applies to a fallback session, plus
  the two-sessions-one-tree hygiene rules below.

**Recon & review subagents** (spawned from the coordinator, no work order needed)
- Read-only recon ("how does X work", codebase sweeps) via cheap search agents.
- Review passes over an executor's diff — the coordinator reads only the findings.
- Micro-tasks too small to justify a work order.

## The work-order contract

Delegation of real implementation always goes through a written work order created from
`work-orders/TEMPLATE.md`. Work orders live in **per-branch subdirectories**:

```
work-orders/<branch-slug>/NNN-slug.md
```

- `<branch-slug>` is the current branch name with `/` replaced by `-` (git: `feature/auth` →
  `feature-auth`; svn: the branch directory name, `trunk` for trunk work).
- `NNN` is zero-padded and sequential **within that branch's subdirectory** — the numbering tells
  the story of what happened on that branch, in order. Different branches never collide, so merges
  never conflict on work-order names; after a merge, `work-orders/` accumulates each branch's
  history, which is the point.

The order carries scope, files in play, acceptance criteria, constraints, and a mandatory
**delivery block** — the per-dispatch decisions, filled in every time rather than remembered:

1. **Vehicle** — executor subagent (default) | fallback executor session | inline (coordinator
   does it; only for trivia below work-order threshold).
2. **Model tier** — workhorse or premium, with a one-line why.
3. **Isolation** — shared working tree (default) or isolated worktree (git only; required for
   parallel orders touching overlapping files).
4. **Scheduling** — serial (default) or parallel with named sibling orders; must respect the
   project's parallel-dispatch default above.

The executor answers with a **result report** appended to the same file: terse, structured,
pointers over payloads — what changed, test/build results, deviations from the order, escalations.
The coordinator reviews from this report.

Work orders and their result reports are **committed to the repo.** They are project
documentation — the record of what was attempted, what worked, and what didn't — and they are the
paper trail the terse-commit rule below leans on.

## Context discipline

- Reports return **pointers + verdicts**, not inline dumps.
- Decisions land in `docs/4-decision-log.md` **as they're made**, not at session end. A coordinator
  reset (crash, clear, compaction) must not lose rationale. See the log-as-you-go standing
  instruction in `AGENTS.md`.
- The project's docs must always be sufficient to reboot a fresh coordinator cheaply. If they
  aren't, that's the bug — fix the docs, not the model.

## Version-control rule (hard)

**The agent never commits without explicit, per-commit human permission — in any role, in any
session or subagent — and never performs history-rewriting or publishing operations.** Its job is
to *remind*: when work is review-ready, the result report (or coordinator summary) says so and
suggests committing. Commit messages are **human-written where possible**; an agent-drafted message
is a suggestion to edit, not an auto-applied message. Rationale: for professional work, commit
authorship and history stay in human hands; the docs, code, and work orders carry any detail a
terser human message omits.

Corollaries:
- An executor ends its work orders at "ready to commit," never at "committed."
- One pair of human hands owns the working tree's VCS state.

### On git

- `push`, `rebase`, and `reset` are denied in tracked `.claude/settings.json` — these never lift.
- `commit` is *not* denied; it stays a per-commit permission prompt, and that prompt is the human
  approval the rule requires.

### On Subversion

- `svn commit` is `git commit` **and** `git push` in one step — there is no local commit; every
  commit publishes to the server immediately. Accordingly, **`svn commit` is denied outright** in
  tracked `.claude/settings.json`: on svn projects the human runs the commit, always. There is no
  such thing as a safe local agent commit on svn — every commit lands on the shared server.
- There is nothing else to deny — push/rebase/reset don't exist as separate svn operations.

### Exception: autonomous-checkpoint mode (opt-in, per-project, local, **git only**)

For **exploratory or throwaway work** where you want the agent to run a long multi-step process
largely unattended — and you want checkpoints to roll back to — you can allow autonomous commits by
adding `Bash(git commit:*)` to `.claude/settings.local.json` (gitignored, so it never ships with
the template or becomes a project default). Push/rebase/reset stay denied, so the checkpoints are
**purely local** — exactly what rollback wants. Use this only on projects unlikely to reach
production in their initial form; never as a default on professional work.

**This mode does not exist on svn projects.** svn has no local commit, so every "checkpoint" would
publish straight to the shared server. The deny rule stays; don't work around it.

## Working-tree hygiene

**Serial dispatch (the default).** One order in flight at a time; coordinator review happens
*after* the executor reports done and *before* the next order dispatches to the same area. The
executor builds/tests to a scratch artifacts path if local processes hold output locks.

**Parallel dispatch (git projects that opted in).** Orders touching **disjoint** file sets may run
as parallel subagents in the shared tree. Orders that might overlap require **worktree isolation**
(the Agent tool's worktree option) so each subagent edits its own copy; the coordinator reconciles
on review. Expect the trade-off the interview explained: more throughput and faster token burn, and
review arrives in batches.

**svn projects are always serial for overlapping work.** Subversion has no worktrees; parallel
subagents would share one working copy. Parallel dispatch is limited to genuinely disjoint file
sets, and when in doubt, serialize.

**Fallback executor sessions** (two sessions, one directory) additionally observe: no concurrent
edits to the same files from two sessions; only the coordinator writes the five docs — the executor
writes code and its result report; the human arbitrates when coordinator review and the next
dispatch would race.

## Failure modes this guards against

- **Premium-model burn on mechanical work** → per-dispatch model tier on executor subagents.
- **Coordinator context bloat → compaction → plan drift** → churn quarantined in disposable
  subagent contexts; docs as rebootable memory.
- **Delegation decisions made from memory mid-flow** → the delivery block is a mandatory
  work-order field, not a disposition to remember.
- **Executor contract drifting between two definitions** → the SKILL.md is the single source of
  truth; the agent definition is a pointer to it.
- **Work-order name collisions across branches** → per-branch subdirectories with per-branch
  numbering.
- **Agent-authored commits in professional history** → hard VCS rule; `svn commit` denied outright.
- **Parallel edits clobbering one tree** → serial default; worktree isolation required for
  overlapping parallel orders; svn serialized.
