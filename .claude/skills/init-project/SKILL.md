---
name: init-project
description: One-time setup for a freshly-cloned project template — interview the developer about the project, then fill in AGENTS.md, the README, and the docs/ baseline so future sessions start oriented. Run once, at the very start.
allowed-tools: Read Glob Grep Edit Write Bash(git status:*) Bash(git log:*)
---

You are initializing a new project created from the agent project template. The repo currently
carries placeholder scaffolds; your job is to **interview the developer** and then **fill those
scaffolds in** so that every future start-session reads accurate, project-specific state.

Work in two phases: **interview first, write second.** Do not edit any file until you've gathered the
answers and played them back.

## 0. Confirm this is an uninitialized template

Read `AGENTS.md`. If the 🌱 *uninitialized template* banner is already gone and the project sections
are filled in, this project has been initialized — stop, say so, and ask whether they really want to
re-run setup (which would overwrite project content). Otherwise continue.

## 1. Interview

Ask the developer about the project conversationally — a few focused questions at a time, not a wall
of them. Lead with the essentials (name + what it is), then go deeper. Cover, at minimum:

- **Name** — the project / product name (and a short namespace or repo slug if relevant).
- **What it is** — what you're building, for whom, and why it exists. The one-paragraph orientation a
  fresh context needs most.
- **Goals & success** — what done/working looks like; the near-term objective vs. the longer arc.
- **Tech stack & shape** — languages, frameworks, runtime, how the repo is structured; anything a
  fresh context should know before touching code.
- **Hard constraints** — cost, timeline, scale, compliance, or other first-class limits.
- **Off-limits / read-only areas** — vendored code, generated files, external systems, anything an
  agent must not modify.
- **External dependencies** — services, APIs, other repos, related systems this leans on.
- **How they want to work** — review/commit habits, environment quirks, anything that should shape
  day-to-day behavior. (User-level preferences already apply — capture only project-specific
  additions here.)
- **Question format** — when the agent needs a decision or clarification, should it ask in
  **text** (numbered questions in the reply body — the default; keeps the surrounding context
  visible while answering) or via **pop-up** prompts (structured pickers like Claude Code's
  AskUserQuestion — faster for discrete yes/no or option choices, but they hide the reply text
  behind the prompt)? Sets the per-project preference in `AGENTS.md`; either way it's a default
  the developer can override in the moment.
- **Version control** — **git or Subversion?** Often this isn't a choice — some organizations and
  project types mandate svn, full stop — so ask which applies rather than assuming git. The answer
  changes real behavior (see `docs/6-working-model.md`): on svn there is no agent commit at all
  (`svn commit` publishes straight to the server and is denied), no autonomous-checkpoint mode,
  and no worktree isolation for parallel work.
- **Working mode** — will this project run **solo** (one session does everything) or
  **coordinator/subagents** (a premium session coordinates — plans, decides, reviews — while
  executor subagents implement via written work orders)? See `docs/6-working-model.md` for the
  distinction. This sets the project's default mode; it's a default, not a lock. Default to
  **solo** unless the developer expects sustained, heavy implementation work that would benefit
  from the split.
- **Parallel dispatch** (only if coordinator/subagents, and effectively git-only) — may the
  coordinator run multiple work orders at once? Don't assume the developer is a coding-agent
  expert; explain the trade-off in plain terms before asking: *parallel gets more done per hour
  and burns tokens faster, and the review work arrives in batches — several changes to check at
  once instead of one at a time. Serial is slower but each result gets reviewed before the next
  starts, which is the safer default if this workflow is new to you or the codebase is
  unfamiliar.* Default to **serial**. On svn projects, record serial without belaboring it —
  Subversion has no worktrees, so parallel overlapping work isn't safe there anyway.
- **Kickoff decisions already made** — any approach/architecture calls already settled, with the why.
  These seed the decision log.

Use judgment — skip what doesn't apply, dig where answers are thin. If the developer points you at
existing material (a brief, a related repo, notes), read it and fold it in rather than re-asking.

## 2. Play back, then write

Summarize what you heard in a few lines and confirm before writing. Then populate:

- **`AGENTS.md`** — fill in *What this project is* and *Facts worth knowing before you touch
  anything*; set the title; set the **Project preference** line in *Asking the developer
  questions* to `text` or `pop-up` from the interview; **remove the uninitialized-template
  banner.** Keep the operating-model sections (how-we-work, session workflow, standing
  instructions, doc hierarchy) intact. Leave the `CLAUDE.md` shell alone — it just imports this
  file.
- **`README.md`** — **replace it.** The template ships with a README that describes the template
  itself, which would otherwise end up as the public face of the new project's repo. Overwrite it
  with a minimal project README generated from the interview: project name as the title, the
  one-paragraph description, tech stack if useful, and a short pointer to `docs/` + `AGENTS.md` for
  the project's working docs. Keep it brief — it can grow later; it just must not describe the
  template.
- **`docs/2-project-status.md`** — replace the "uninitialized" block with a real "Where we are right
  now" (project kickoff state), and set "Active now" to the genuine first actions. Stamp today's date.
- **`docs/3-project-roadmap.md`** — fill in *Framing* and a first phase/workstream outline from the
  goals. Stamp the date.
- **`docs/4-decision-log.md`** — write the first dated entry (or entries) capturing the kickoff
  decisions and their rationale, in the file's append-only style.
- **`docs/5-deferred-items.md`** — add any consciously-punted items surfaced in the interview (only if
  real); otherwise leave the scaffold.
- **`docs/6-working-model.md`** — fill in the **Project defaults** block from the interview:
  mode (`solo` or `coordinator/subagents`), version control (`git` or `svn`), and parallel
  dispatch (`serial` or `parallel-allowed`). Leave the rest of the doc as-is (it's the standing
  model, not project-specific). If the project is solo, a one-line note that the
  coordinator/subagents option remains available is enough.
- **`docs/1-documentation-index.md`** — flip operating-set rows (including `6-working-model.md`)
  from 🌱 Scaffold to ✅ Verified as they fill in, stamp dates, and add rows for any
  project/reference docs the developer pointed at.
- **`.claude/settings.json`** (Claude Code only — skip in other agents) — if the developer named
  off-limits/read-only areas, propose adding `deny` rules for those paths (confirm the absolute paths
  first). Add any safe project-specific `allow` entries they want.

## 3. Report

Summarize what you filled in and flag anything still thin (a section you couldn't complete from the
interview). Suggest the developer run start-session next session to confirm the orientation reads
cleanly. Don't commit unless asked.

> Use today's date for all stamps. Preserve each document's established voice — these are read cold by
> future contexts, so consistency matters.
