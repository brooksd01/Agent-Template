# Agent Project Template

A starting skeleton for new projects worked on with AI coding agents — Claude Code, Codex CLI,
Cursor, Gemini CLI, Copilot, or anything else that reads the [AGENTS.md](https://agents.md)
convention. It gives a fresh project two things out of the box: a **documentation structure** that
keeps a project's state legible to a fresh AI context, and a set of **session workflow skills**
that maintain that structure as you work.

The idea: an agent session starts with no memory of yesterday. Rather than rely on it to
semantically search your docs (it can't), this template keeps a small, deliberately-maintained set
of documents that *are* the project's memory — and workflows that read them at the start of a
session and write them back at the end.

> **Note:** this README describes the template. When you initialize a project from it,
> `/init-project` replaces this file with a README for *your* project.

## How it works

### The orientation files

- **`AGENTS.md`** — the canonical, tool-agnostic project orientation, read natively by most coding
  agents. Auto-loads (or is read first) every session and points at the docs below.
- **`CLAUDE.md`** — a thin Claude Code shell: imports `AGENTS.md` via `@AGENTS.md`, then adds only
  Claude Code-specific notes (permission discipline, hook behavior). Edit `AGENTS.md` for anything
  every tool should know.

### The documents (`docs/`)

A fixed five-document operating set, plus room to grow:

| File | Horizon | Role |
|---|---|---|
| `docs/1-documentation-index.md` | **Map** | Catalogue of every doc, its purpose, and its status. The first thing to consult — don't guess at filenames. |
| `docs/2-project-status.md` | **Now** | Living "where we are / what's next / what's blocked." Read first, update last. |
| `docs/3-project-roadmap.md` | **Strategic** | Phase/workstream plan and sequencing. |
| `docs/4-decision-log.md` | **Permanent** | Append-only, dated decisions *and their rationale*. Never rewritten. |
| `docs/5-deferred-items.md` | **Someday** | Consciously-punted items, each with a revisit trigger. |

Project-specific documents (reference material, design drafts, specs) get added to `docs/` over time
and catalogued in the index so a fresh context can find them.

### The session workflow (`.claude/skills/`)

Four skills in the open [Agent Skills](https://agentskills.io) format (`SKILL.md`), which is read
by Claude Code and 30+ other tools. In Claude Code they're slash commands; in other agents,
`AGENTS.md` instructs the agent to read and follow the relevant `SKILL.md` when you ask for it by
name.

| Skill | When | What it does |
|---|---|---|
| `init-project` | Once, at the very start | Interviews you about the project, then fills in `AGENTS.md`, this README, and the `docs/` baseline. |
| `start-session` | Start of each session | Reads the docs + git state, briefs you on where things stand (and offers to reconcile drift), then waits. |
| `save-context` | Mid-session checkpoint | Freshens STATUS only. Safe to run repeatedly. |
| `end-session` | End of a session | Full close-out: reconciles STATUS, appends decisions, records deferrals, sweeps for newly-stale docs, stamps the index. |

The loop is: start-session to orient → work → save-context to checkpoint → end-session to write
everything back so the next session starts accurate. Two backstops reduce the cost of forgetting
the loop: `AGENTS.md` tells the agent to proactively offer the close-out when you signal wrap-up or
context runs low, and (in Claude Code) a `PreCompact` hook reminds it to checkpoint STATUS before
compaction.

### Coordinator / executor working model (optional)

For projects with sustained implementation work, the template supports a two-tier model: a
premium, high-reasoning model **coordinates** (plans, decides, reviews) in one long-lived session,
while cheaper **executor** sessions and subagents do the implementation — each driven by a written
**work order** in `work-orders/`. This spends premium capacity only on thinking and keeps the
coordinator's context light and long-lived, with the docs as rebootable memory if it's ever reset.

- `docs/6-working-model.md` defines the model, the work-order contract, and a hard git rule
  (the agent never commits without per-commit permission, and never pushes/rebases/resets —
  enforced by `deny` rules in `.claude/settings.json`).
- `/dispatch-work-order` (coordinator) drafts an order; `/execute-work-order NNN` (executor) runs
  one in a separate session and reports back.
- It's **opt-in per project** — `/init-project` records whether a project defaults to solo or
  coordinator/executor, and the skills simply go unused on solo projects.

### Why not just the agent's built-in memory?

Claude Code and other tools now ship machine-local auto-memory. That solves a different problem:
it's per-machine, per-user, and per-tool. This template's docs are **committed to the repo** —
shared across the team, portable across tools, reviewable in PRs, and part of the project's history.
The two coexist fine.

## Getting started

1. **Get a copy.** Clone or copy this folder for your new project. (Each new project is its own copy —
   the template is reused by re-cloning, not shared.)
2. **Open it in your agent.** `AGENTS.md` (via `CLAUDE.md` in Claude Code) loads and shows a 🌱
   *uninitialized template* banner.
3. **Run `/init-project`** (or ask the agent to run the init-project workflow). It interviews you
   about the project — what it is, goals, tech stack, hard constraints, off-limits areas — then
   fills in `AGENTS.md` and the five docs, replaces this README with your project's, writes the
   first decision-log entry, and removes the banner.
4. **From then on,** start each working session with `/start-session` and close it with
   `/end-session` (or `/save-context` for a quick mid-session save).

That's it — the project is initialized and the docs carry its memory forward.

## Notes

- **Permissions (Claude Code).** Safe, reusable allowances (read-only git inspection) live in
  tracked `.claude/settings.json`. Machine-local or secret overrides go in
  `.claude/settings.local.json`, which is gitignored. As your common commands settle, run
  `/fewer-permission-prompts` to extend the allowlist. Other tools bring their own permission
  systems.
- **Git.** This template ships without git history. Run `git init` in your copy and commit when
  you're ready.
