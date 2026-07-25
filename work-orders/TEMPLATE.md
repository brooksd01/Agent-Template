# WO-NNN — [slug / short title]

> Copy this file to `work-orders/NNN-slug.md` (zero-padded number, next in sequence). The
> coordinator fills in the **Order** half; the executor (or subagent) appends the **Result report**
> half. Both halves are committed — this file is the record of what was attempted and how it went.
> See `docs/6-working-model.md` for the full contract.

**Status:** _drafted → dispatched → in progress → ready to commit → reviewed → done_
**Created:** _[date]_ · **Coordinator:** _[session/model]_

---

## Order (coordinator writes)

### Delivery vehicle _(mandatory)_
- **Vehicle:** _executor session_ | _subagent_
- **Justification (one line):** _[why this vehicle — e.g. "build-test-fix loop → executor";
  "read-only recon → subagent"]_

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
dependencies, performance/compat limits. Also: what to escalate rather than decide.]_

---

## Result report (executor / subagent appends)

**Outcome:** _completed_ | _partial_ | _blocked_ — one line.

### What changed
_[Files touched and the shape of the change. Pointers, not diffs.]_

### Build / test results
_[Commands run and their verdicts. Point to a scratch log path rather than pasting output.]_

### Deviations from the order
_[Anything done differently than specified, and why. "None" is a valid answer.]_

### Open questions / escalations
_[Decisions surfaced back to the coordinator; ambiguities hit; follow-ups. "None" is valid.]_

### Ready to commit?
_[Yes/no. This report ends at "ready to commit" — never at "committed." The coordinator or human
owns the commit decision.]_
