# Documentation Index

The map of everything in `docs/`. This index is how a fresh context discovers what exists and decides
what to read. **Consult this before opening docs by guesswork.**

**Last updated:** _[stamped by init-project]_

## How to read the Status column

| Mark | Meaning |
|---|---|
| ✅ **Verified** | Current and confirmed accurate; actively relied on. |
| ⚠️ **Suspect** | Predates later decisions; partly superseded. Verify before relying. |
| 🔴 **Obsolete** | Superseded; retained only for history. |
| 📚 **Historical** | Accurate record of completed work, but not an active spec. |
| 🌱 **Scaffold** | Newly created placeholder; structure in place, content still thin. |

---

## ▶ Start here (load-bearing for a fresh context)

1. **`../AGENTS.md`** — project orientation and operating model (auto-loaded; Claude Code loads it via the `CLAUDE.md` shell).
2. **`2-project-status.md`** — where we are right now, what's next, what's blocked.
3. **`3-project-roadmap.md`** — strategic phase plan.
4. **`4-decision-log.md`** — decisions + rationale; the most authoritative file.
5. **`6-working-model.md`** — the session/working model and the git rule; read if this project uses the coordinator/executor split.

---

## Operating set (the five numbered docs)

| Document | Purpose | Status | Last Updated |
|---|---|---|---|
| `1-documentation-index.md` | This file — catalogue of all documentation. | 🌱 Scaffold | — |
| `2-project-status.md` | Living current-state, near-term checklist, blockers. The day-to-day surface. | 🌱 Scaffold | — |
| `3-project-roadmap.md` | Strategic phase/workstream plan — sequencing and risks. | 🌱 Scaffold | — |
| `4-decision-log.md` | Append-only, dated decisions with rationale. | 🌱 Scaffold | — |
| `5-deferred-items.md` | Consciously-punted items, each with a revisit trigger. | 🌱 Scaffold | — |
| `6-working-model.md` | How sessions are structured (solo vs. coordinator/executor) + the hard git rule. | 🌱 Scaffold | — |

---

## Project documents (added as the project grows)

Beyond the five-doc operating set, project-specific documents live in `docs/` and get catalogued here
so a fresh context can find them. Add a row per doc with its purpose and status. Common categories:

- **Reference / input material** — processed extractions of source material (requirements, glossaries,
  sample inputs) kept as maintained working copies. Mark them as *seeds* until verified.
- **Design drafts** — first-pass design artifacts (architecture, interface/contract sketches) expected
  to evolve.
- **Specs** — settled contracts and specifications, once they firm up.

_(No project documents yet — init-project and ongoing work populate this.)_

**`work-orders/`** — for coordinator/executor projects, the numbered work orders and their result
reports (`NNN-slug.md`, from `TEMPLATE.md`). Committed as project history: the record of what each
delegated task scoped, and what worked or didn't in the build. See `docs/6-working-model.md`.

---

## External references (added as needed)

Point here to load-bearing material that lives outside `docs/` — source codebases the project builds
on, related repos, read-only reference trees, external systems. Note each one's location, purpose, and
whether it's authoritative or historical.

_(None yet.)_
