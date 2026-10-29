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
- **Data access & sensitivity** — skip if the project touches no real data. Otherwise: what data
  is involved (databases, customer records, production systems); what the agent may see (full
  rows, or only counts and aggregates); how it gets access (ideally a dedicated, least-privilege
  login and a read-only query path); where it may write, and what's strictly read-only; and whether
  outbound side effects — email, webhooks, third-party API calls — must be fenced off outside
  production. If the agent will need data access to do the work, suggest arranging it early:
  analysis done from code alone, before real data is available, tends to get redone.
- **How they want to work** — review/commit habits, environment quirks, anything that should shape
  day-to-day behavior. (User-level preferences already apply — capture only project-specific
  additions here.)
- **Version control** — **git or Subversion?** Often this isn't a choice — some organizations and
  project types mandate svn, full stop — so ask which applies rather than assuming git. The answer
  changes real behavior (see `docs/6-working-model.md`): on svn there is no agent commit at all
  (`svn commit` publishes straight to the server and is denied), no autonomous-checkpoint mode,
  and no worktree isolation for parallel work.
- **Repository layout** — **single** (the default: code and these docs share one repo) or
  **split** (this repo becomes the *harness* — docs, work orders, agent tooling — and the code
  lives in one or more separate repos cloned under `repos/<name>/`). Explain the trade-off in plain
  terms: *split keeps the code repo free of agent tooling, so later developers can bring their own
  setup or none, and lets one harness drive several repos (say, web and mobile). The cost is two
  repos to commit to, a little cross-referencing between them, and no parallel work in
  overlapping code.* Default to **single** unless the developer wants the code repo kept clean or
  expects several code repos. If split, for each code repo: a short name, the clone URL, its VCS,
  its default branch, and whether it already exists or is new. Ask the harness's own VCS
  separately (recommend git — it can be git even when a code repo must be svn).
- **Working mode** — will this project run **solo** (one session does everything) or
  **coordinator/subagents** (one long-lived session coordinates — plans, decides, reviews — while
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
- **Model tiers** — which model runs each role (see "Model tiers" in `docs/6-working-model.md`).
  Propose rather than interrogate: the model *this* session is running on is the likely
  coordinator, so name it and offer the defaults in one question — e.g. *"This session is running
  on Opus. Is that what you'll normally coordinate with? For implementation I'd suggest the same
  model for most work and Sonnet for easier or mechanical tasks — want anything different?"*
  Defaults when the developer has no opinion: **coordinator** = this session's model;
  **standard executor** = the same; **light executor** (also recon/search subagents) = `sonnet`.
  If you can't tell what model you're running on, ask. Record Claude Code aliases (`fable`,
  `opus`, `sonnet`, `haiku`), not version numbers. Solo projects need only the coordinator and
  light rows (solo sessions still spawn recon subagents); fill standard with the coordinator's
  model.
- **Kickoff decisions already made** — any approach/architecture calls already settled, with the why.
  These seed the decision log.

Use judgment — skip what doesn't apply, dig where answers are thin. If the developer points you at
existing material (a brief, a related repo, notes), read it and fold it in rather than re-asking.

## 2. Play back, then write

Summarize what you heard in a few lines and confirm before writing. Then populate:

- **`AGENTS.md`** — fill in *What this project is* and *Facts worth knowing before you touch
  anything* (including the data-access answers, if any); set the title; **remove the
  uninitialized-template banner.** Keep the operating-model sections (how-we-work, session
  workflow, standing instructions, doc hierarchy) intact. Leave *Standing rules* at "None yet"
  unless the interview produced hard, checkable limits — e.g. "aggregates only, never client rows"
  or "never write to the production database" — in which case propose them as the first rules and
  log them with the kickoff decisions. Leave the `CLAUDE.md` shell alone — it just imports this
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
  mode (`solo` or `coordinator/subagents`), layout (`single` or `split`), version control (`git`
  or `svn`), parallel dispatch (`serial` or `parallel-allowed`), the **Code repositories** table
  (split only — delete it on single-repo projects), and the **Model tiers** table (replace the
  `e.g.` placeholders with the chosen aliases). Leave the rest of the doc as-is (it's the standing
  model, not project-specific). If the project is solo, a one-line note that the
  coordinator/subagents option remains available is enough.
- **`.claude/agents/executor.md`** (Claude Code only) — set the `model:` frontmatter line to the
  **standard executor** alias from the tier table. Change nothing else in that file.
- **`.gitignore`** — the template ships with only the agent-local settings entry. Add the standard
  ignores for the project's stack (build output, dependency directories, IDE folders, local env
  files). Keep the `.claude/settings.local.json` line. On a **split** project, the harness holds
  no code: add `/repos/` instead of the stack ignores (those belong in each code repo).
- **Split layout only:**
  - **`AGENTS.md` → *Repository layout*** — fill in the one-line-per-repo summary and keep the
    split-layout rules. (On a single-repo project, replace the placeholder with "This repo holds
    both the code and the project docs." and delete the split-layout rules block.)
  - **Clone the code repos** into `repos/<name>/` — run the clones with the developer's
    permission, or give them the commands to run. A brand-new code repo is the developer's to
    create on its host first.
  - **Draft `AI-HARNESS.md` at each code repo's root** — a short pointer back to this harness:
    the harness repo's URL, today's date and the harness commit at setup (if there is one yet), one
    line on what the harness holds (planning docs, decision log, work orders), a note that
    contributors don't need it to work on the code, and where the code's own architecture and
    setup docs live. Leave it **uncommitted** in the code repo — the developer commits it there.
  - **`README.md`** (the new project README) — include getting-started steps for a fresh machine:
    clone the harness, then clone each code repo into `repos/<name>/`, then launch the agent from
    the harness root.
- **`docs/1-documentation-index.md`** — flip operating-set rows (including `6-working-model.md`)
  from 🌱 Scaffold to ✅ Verified as they fill in, stamp dates, and add rows for any
  project/reference docs the developer pointed at.
- **`.claude/settings.json`** (Claude Code only — skip in other agents) — if the developer named
  off-limits/read-only areas, propose adding `deny` rules for those paths (confirm the absolute paths
  first; on a split project they'll usually sit under `repos/<name>/`). Add any safe project-specific `allow` entries they want.

## 3. Report

Summarize what you filled in and flag anything still thin (a section you couldn't complete from the
interview). Suggest the developer run start-session next session to confirm the orientation reads
cleanly. Don't commit unless asked.

> Use today's date for all stamps. Preserve each document's established voice — these are read cold by
> future contexts, so consistency matters.
