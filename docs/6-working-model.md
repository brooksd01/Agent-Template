# Working Model — Coordinator / Subagents

**Purpose:** How agent sessions are structured on this project. Two operating modes: **solo**
(one session does everything) and **coordinator/subagents** (one long-lived session coordinates;
subagents execute work orders). This doc defines the coordinator/subagent model and the rules that
hold whichever mode is in play. `init-project` records the project's defaults below.

**Project defaults** _(set by init-project)_:

- **Mode:** _[`solo` or `coordinator/subagents`]_
- **Layout:** _[`single` (default — the code lives in this repo) or `split` — this repo is the
  harness and the code lives in separate repos under `repos/`. See "Split layout."]_
- **Version control:** _[`git` or `svn` — on split projects, this repo's (the harness's); each
  code repo's is in the table below]_
- **Parallel dispatch:** _[`serial` (default) or `parallel-allowed` — git projects only; svn is
  always serial for overlapping work, and so is overlapping work in a split project's code repos.
  See "Working-tree hygiene."]_
- **Code repositories** _(split layout only — delete on single-repo projects)_:

  | Name | Path | VCS | Remote | Default branch |
  |---|---|---|---|---|
  | _[e.g. `web`]_ | _[`repos/web/`]_ | _[`git`]_ | _[clone URL]_ | _[`main`]_ |
- **Model tiers:** _[one Claude Code alias per role — see "Model tiers" below]_

  | Role | Covers | Model |
  |---|---|---|
  | **Coordinator** | The main session — planning, decisions, review | _[e.g. `opus`]_ |
  | **Standard executor** | Most implementation work orders | _[e.g. `opus`]_ |
  | **Light executor** | Mechanical / well-specified orders; recon and search subagents | _[e.g. `sonnet`]_ |

> These are **defaults, not locks.** A solo-default project can still dispatch a work order for one
> heavy task; a coordinator project can still do small things inline; a serial project can run one
> deliberate parallel batch. The defaults just tell a freshly-booted session what to expect.

---

## Why this model

The primary goal is **context isolation**: keep the coordinator's context light and long-lived by
pushing all detail churn — file reads, build output, fix loops — into disposable subagent contexts.
The five-doc system is what makes the coordinator *cheaply rebootable* — long-lived is the
optimization, cheaply-reborn is the fallback, and the docs are the fallback's fuel.

The secondary goal is **cost**: mechanical work can run on a cheaper model tier. This matters less
than it once did — adjacent model tiers are now only a few times apart in price, and a capable
model often finishes a task in fewer turns — so many projects run their standard executor on the
same model as the coordinator and reserve the light tier for genuinely easy work.

## Model tiers

The project's tier table lives in **Project defaults** above, and is the one place model choice is
recorded. Everything else — work orders, the dispatch skill — refers to tiers by role
(`coordinator-tier`, `standard`, `light`), never by model name, so a model change is a one-line
edit here.

- Models are recorded as **Claude Code aliases** (`fable`, `opus`, `sonnet`, `haiku`), which track
  the latest release in each family, so new versions flow in without doc edits. In other tools, use
  that tool's equivalent tier.
- The coordinator is whatever model the main session runs on; init-project proposes the model it
  was itself run on.
- `.claude/agents/executor.md` declares the **standard** tier as its default `model:`, so a dispatch
  that forgets to name a model falls back to the standard tier rather than silently inheriting the
  coordinator's (which can be the most expensive tier). The `model` passed on each Agent call
  overrides it.
- Three roles is deliberate. Add a separate recon tier only if a cheaper model class becomes worth
  routing to on its own.
- The table is a **project** default, shared by everyone on it. Anyone can deviate for a given
  dispatch; change the table only when the project's default should change.

## Roles

**Coordinator** (coordinator-tier model; one long-lived session — the normal `/start-session`
session)
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
- Model tier is a **per-dispatch decision**: the coordinator picks a tier in the delivery block
  and passes the matching alias from the tier table as `model` on the Agent call (`light` for
  mechanical orders, `standard` for most, `coordinator-tier` for genuinely hard ones).

**Fallback: executor session** (a separate human-opened session, same working tree)
- The same contract, delivered by hand: open a session, run `/execute-work-order <path>` (or point
  any other agent at the same SKILL.md). Use it when the tool in play lacks subagent support, or
  for an unusually large order that wants an interactive, long-running session.
- Everything in this doc that applies to executor subagents applies to a fallback session, plus
  the two-sessions-one-tree hygiene rules below.

**Recon & review subagents** (spawned from the coordinator, no work order needed)
- Read-only recon ("how does X work", codebase sweeps) via search agents on the light tier.
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
- **Split layout:** orders file under the code repo they target, keyed to *that* repo's branch —
  `work-orders/<repo>/<branch-slug>/NNN-slug.md`. See "Split layout" below.
- `NNN` is zero-padded and sequential **within that branch's subdirectory** — the numbering tells
  the story of what happened on that branch, in order. Different branches never collide, so merges
  never conflict on work-order names; after a merge, `work-orders/` accumulates each branch's
  history, which is the point.

The order carries scope, files in play, acceptance criteria, constraints, the earlier orders it
**depends on**, and a mandatory **delivery block** — the per-dispatch decisions, filled in every
time rather than remembered:

1. **Vehicle** — executor subagent (default) | fallback executor session | inline (coordinator
   does it; only for trivia below work-order threshold).
2. **Model tier** — `light`, `standard`, or `coordinator-tier` (resolved through the tier table
   above), with a one-line why.
3. **Isolation** — shared working tree (default) or isolated worktree (git only; required for
   parallel orders touching overlapping files).
4. **Scheduling** — serial (default) or parallel with named sibling orders; must respect the
   project's parallel-dispatch default above.

While it works, the executor appends milestones to the order's **progress log**, so a run cut off
mid-order (usage limit, crash, timeout) resumes from the last entry on re-dispatch instead of
restarting. It finishes with a **result report** appended to the same file: terse, structured,
pointers over payloads — what changed, test/build results, deviations from the order, escalations,
plus two sections that carry work forward:

- **Shortfall** — what didn't land, stated honestly even on a "completed" order. The coordinator
  routes each item to STATUS, a follow-up order, or deferred items, so nothing unfinished lives only
  in a report.
- **Notes for later orders** — patterns, helpers, and traps a later order must know. Binding on
  any order that lists this one under **Depends on**; the executor of that order reads them at boot.

The coordinator reviews from this report.

**Standing rules** (in `AGENTS.md`) bind every order without being restated in it. When a review
turns up a fix for a whole class of bug rather than one instance, the coordinator proposes a
standing rule guarding against the class; with the human's approval it's added and logged. That's
how a lesson learned in one order is enforced in every later one.

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
  The deny rules also cover the forms with options before the subcommand (`git -C <path> push`),
  which a plain `git push` rule doesn't match. Claude Code's docs are explicit that command-text
  rules aren't a hard security boundary; they're a guard against agent drift, and a PreToolUse
  hook is the escalation if they ever prove insufficient.
- `commit` is *not* denied; it stays a per-commit permission prompt, and that prompt is the human
  approval the rule requires.

### On Subversion

- `svn commit` is `git commit` **and** `git push` in one step — there is no local commit; every
  commit publishes to the server immediately. Accordingly, **`svn commit` is denied outright** in
  tracked `.claude/settings.json` — including its built-in alias `svn ci` and the forms with
  options before the subcommand: on svn projects the human runs the commit, always. There is no
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

## Split layout

A project can keep its harness — these docs, the work orders, `.claude/`, `AGENTS.md` — in one
repo and its code in others. The code repos stay free of agent tooling, so later developers can
use their own setup or none, and one harness can drive several code repos (web, mobile, API).
`AGENTS.md` carries the day-to-day rules (where to launch, which docs live where, whose
instructions win); this section covers how the working model adapts.

**Shape.** Each code repo is cloned into `repos/<name>/` inside the harness — always one folder per
repo, even when there's only one — and the harness gitignores `repos/`. The **Code repositories**
table in Project defaults is the one place skills look up each repo's path, VCS, remote, and
default branch.

**Work orders.** An order targets one code repo, named in its **Repo** field, and files under
`work-orders/<repo>/<branch-slug>/` using that repo's branch. Numbering is per repo and branch. An
order that genuinely must touch two repos files under its main one and lists the other in Files in
play — but prefer splitting it in two.

**Linking commits across repos.** A work order and the code it produced land in different repos,
so tie them together by convention:
- An agent-drafted commit message for a code repo includes the work-order reference (e.g.
  `WO web/feature-auth/003`).
- After the human commits, the coordinator records the code commit (repo + hash) on the order's
  **Commits** line.
- The harness's own changes — reports, STATUS, the decision log — are committed separately in the
  harness.

**Parallel work.** Claude Code's worktree isolation creates worktrees from the repo it was launched
in, with tracked files only — so a gitignored code repo isn't in them at all. Overlapping work in a
code repo is therefore **serial**, as on svn; parallel dispatch is limited to genuinely disjoint
file sets.

**Checkpoint commits.** In autonomous-checkpoint mode, checkpoints land in each git repo touched —
code repo and harness — and rolling back means rolling back both to matching points. (A code repo
on svn gets no agent commits, checkpoint mode or not.)

## Unattended runs

By default the agent asks when it hits a decision and waits for the answer. For a long run while
the human is away — overnight, or across days — that default stalls the run on the first
question. An **autonomy envelope** changes it for one run. The human grants it explicitly, in
conversation, for that run; it's never a project default, and it ends when the human is back.

**Before launch, settle the forks.** Every decision made before the run starts is one it can't
stall on or guess wrong about: stack choices, scope in and out, feature dispositions. Spend the
setup time there.

**The envelope states, and STATUS records while it's active:**

- **Window** — how long the human will be away, and when they'll next check in.
- **Real blockers** — what still stops the run and waits for the human. At minimum: anything
  irreversible or destructive, anything that would cross the version-control rule, a deny rule, a
  standing rule, or an off-limits area, and any fork where a wrong default would mean redoing a
  large amount of work. The human can add to this list, but not remove those minimums.
- **Parking** — everything short of a real blocker gets the most conservative reasonable default
  and keeps moving. Each is recorded under **Parked (default applied)** in STATUS: the question,
  the default taken, and where it was applied, so a reversal is cheap to find.
- **Budget** — the token/usage stance for the run (no constraint, or a cap). A usage-limit
  interruption isn't a failure: executors' progress logs let the run resume where it stopped.
- **Commits** — whether autonomous-checkpoint mode is on for this run (git only; see the exception
  above). Without it, work accumulates uncommitted until the human is back, so keep the result
  reports clear enough to review a batch at once.

**Parked defaults are provisional, not decisions.** They don't go in the decision log until the
human confirms them.

**When the human is back,** the envelope ends. They answer the Parked list as one batch:
confirmed defaults get logged as decisions; reversed ones become actions in STATUS. Remove the
envelope marker from STATUS, and if checkpoint mode was enabled just for this run, revoke it.

The envelope works in both modes, but it's mostly used for coordinator runs, where executors
already work without asking (see **Executor subagents** above). Nothing in it relaxes the
version-control rule, deny rules, or standing rules.

## Failure modes this guards against

- **Top-tier model burn on mechanical work** → per-dispatch model tier on executor subagents, and a
  standard-tier default on the executor so a forgotten `model` doesn't inherit the coordinator's.
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
- **An interrupted executor restarting from scratch** → the progress log; re-dispatch resumes from
  its last entry.
- **Unfinished work vanishing into "completed" reports** → a mandatory Shortfall section, routed
  by the coordinator on review.
- **The same lesson relearned order after order** → Notes for later orders (binding on dependents)
  and standing rules (binding on everything).
- **An unattended run stalling on its first question — or guessing silently** → an explicit
  autonomy envelope: real blockers stop the run; everything else is parked with a recorded default.
- **A push slipping past the deny rules as `git -C <path> push`** → deny rules for the
  option-before-subcommand forms (`git * push`), which split projects hit routinely.
- **Launching Claude inside a code repo and silently losing every guardrail** → split projects
  always launch from the harness root, where the settings and hooks live.
- **A code change and its work order drifting apart across repos** → the work-order reference in
  the code commit message, and the code commit recorded back on the order.
