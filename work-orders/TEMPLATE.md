# WO-NNN — [slug / short title]

> Copy this file to `work-orders/<branch-slug>/NNN-slug.md` — where `<branch-slug>` is the current
> branch with `/` → `-` (svn: the branch directory name, `trunk` for trunk work) and `NNN` is
> zero-padded, sequential **within that branch's subdirectory**. The coordinator fills in the
> **Order** half; the executor appends the **Result report** half. Both halves are committed — this
> file is the record of what was attempted and how it went. See `docs/6-working-model.md` for the
> full contract.

**Status:** _drafted → dispatched → in progress → ready to commit → reviewed → done_
**Created:** _[date]_ · **Branch:** _[branch]_ · **Coordinator:** _[session/model]_

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
subagent can't ask mid-run; say whether to stop or proceed conservatively when it hits one.]_

---

## Result report (executor appends)

**Outcome:** _completed_ | _partial_ | _blocked_ — one line.

### What changed
_[Files touched and the shape of the change. Pointers, not diffs.]_

### Build / test results
_[Commands run and their verdicts. Point to a scratch log path rather than pasting output.]_

### Deviations from the order
_[Anything done differently than specified, and why. "None" is a valid answer.]_

### Open questions / escalations
_[Decisions surfaced back to the coordinator; ambiguities hit and whether you stopped or proceeded
conservatively; follow-ups. "None" is valid.]_

### Ready to commit?
_[Yes/no. This report ends at "ready to commit" — never at "committed." The human owns the commit;
on svn the agent cannot commit at all.]_
