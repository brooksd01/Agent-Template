# WO-NNN — [slug / short title]

> Copy this file to `work-orders/<branch-slug>/NNN-slug.md` — where `<branch-slug>` is the current
> branch with `/` → `-` (svn: the branch directory name, `trunk` for trunk work) and `NNN` is
> zero-padded, sequential **within that branch's subdirectory**. On a split-layout project, use
> `work-orders/<repo>/<branch-slug>/NNN-slug.md`, with the branch of the code repo the order
> targets. The coordinator fills in the
> **Order** half; the executor keeps the **Progress log** while working and appends the **Result
> report** half. All of it is committed — this file is the record of what was attempted and how it
> went. See `docs/6-working-model.md` for the full contract.

**Status:** _drafted → dispatched → in progress → ready to commit → reviewed → done_
**Created:** _[date]_ · **Branch:** _[branch]_ · **Coordinator:** _[session/model]_
**Repo:** _[split layout only: the target code repo's name from the Code repositories table, e.g.
`web` — delete this line on single-repo projects]_
**Depends on:** _[earlier orders whose "Notes for later orders" bind this one — e.g. `003, 007`
on this branch, or `feature-auth/003` on another — or "none"]_

---

## Order (coordinator writes)

### Delivery block _(mandatory — fill every line, every time)_
- **Vehicle:** _executor subagent_ (default) | _fallback executor session_ | _inline_
- **Model tier:** _light_ | _standard_ (default) | _coordinator-tier_ — _[one line why]_ (alias
  from the tier table in `docs/6-working-model.md`)
- **Isolation:** _shared tree_ (default) | _isolated worktree_ (git only; required if this order
  runs parallel to another touching overlapping files)
- **Scheduling:** _serial_ (default) | _parallel with WO-NNN, …_ — must respect the project's
  parallel-dispatch default in `docs/6-working-model.md`

### Scope
_[What this work order covers, in a sentence or two. What "done" means at the outcome level.]_

### Files in play
_[The files/areas expected to change or be read. Bounds the work and flags overlap with any other
in-flight order touching the same area.]_

### Acceptance criteria
- [ ] _[Concrete, checkable — "tests pass", "endpoint returns X", "no new lint errors"]_
- [ ] _[…]_

### Constraints
_[Anything the executor must respect: don't touch area Y, match existing pattern Z, no new
dependencies, performance/compat limits. Also: what to escalate rather than decide — remember a
subagent can't ask mid-run; say whether to stop or proceed conservatively when it hits one. The
project's standing rules (`AGENTS.md`) apply to every order — don't restate them; list only what's
specific to this one.]_

---

## Progress log (executor appends while working)

_[One short dated line per milestone, appended as it happens — "schema migration added, tests
green", "list page done; starting edit form". If this section already has entries when you start,
an earlier run was interrupted: resume from the last entry rather than restarting.]_

---

## Result report (executor appends)

**Outcome:** _completed_ | _partial_ | _blocked_ — one line.

### What changed
_[Files touched and the shape of the change. Pointers, not diffs.]_

### Build / test results
_[Commands run and their verdicts. Point to a scratch log path rather than pasting output.]_

### Deviations from the order
_[Anything done differently than specified, and why. "None" is a valid answer.]_

### Shortfall
_[What the order asked for that didn't land, or landed weaker than specified — stated plainly, even
if the outcome is "completed". The coordinator routes these to STATUS or deferred items, so an
honest list here is worth more than a clean-looking report. "None" is valid only if it's true.]_

### Notes for later orders
_[Anything a later order building on this one must know or obey — a pattern established, a helper
to use, a trap found. Binding on any order that lists this one under "Depends on". "None" is
valid.]_

### Open questions / escalations
_[Decisions surfaced back to the coordinator; ambiguities hit and whether you stopped or proceeded
conservatively; follow-ups. "None" is valid.]_

### Ready to commit?
_[Yes/no. This report ends at "ready to commit" — never at "committed." The human owns the commit;
on svn the agent cannot commit at all.]_

**Commits:** _[split layout only — the coordinator fills this in after the human commits: repo +
commit hash (or svn revision) for each code repo touched. Leave blank in the executor's report.]_
