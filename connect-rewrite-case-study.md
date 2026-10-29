# Case Study: Agent-Driven Rewrite of USDEC Connect (CFML → .NET 10)

**Purpose of this document:** a dense, sourced account of what was built, how, and what it cost.
It's written as context for other LLM sessions planning similar work, and as the base for human
presentations. A companion PDF (`connect-rewrite-case-study.pdf`, from the `.html` source) covers
the same ground for people, with diagrams.

**As of:** 2026-10-27. Covers 2026-10-08 → 2026-10-14. Repo `F:\Development\USDEC-Connect`, HEAD
`8897415`.
**Status of the work described:** first-pass build complete; **not yet human-reviewed or tested by
users**. The main open planning item is the legacy integrations: the other systems that use the
legacy databases (see §9).

---

## 1. TL;DR

- **The target.** usdecconnect.org: a ~2005 ColdFusion CRM built by Results Direct (RD) for the
  U.S. Dairy Export Council (USDEC) on RD's legacy CMS+ platform.
  - Published Connect: **653 CFML files / 60.6k non-blank lines**.
  - The platform underneath: **~356k lines** of Core plus USDEC client CFML.
  - The data: **493 legacy tables / 23.1M rows / 5.2 GB**.
- **The result.** A working rebuild on **.NET 10, Razor Pages and Dapper** over a greenfield SQL
  Server database.
  - 11 functional areas.
  - **47 of the 58 contract screens**, plus a **22-screen Admin area** the old system never had.
  - **116 routable pages**.
  - **~67k non-blank lines of production code and ~32k of test code**, with **3,918 passing tests**.
- **The data.**
  - **88 legacy tables** (4.16M rows) were read by **73 idempotent load scripts**.
  - They landed **~800k rows** in **103 new tables**.
  - A from-empty rebuild of the whole database runs in **~14 minutes**.
- **The time.**
  - About **2.5 calendar days** from first survey to first-pass complete.
  - The automated build run itself took **31.6 hours wall-clock** and ran largely unattended,
    including overnight.
- **The human effort.**
  - About **8 hours of Brooks's time before launch.**
    - About 2 hours of environment prep.
    - About 6 hours across five setup, priming and planning sittings. That's an upper bound: the
      span of each sitting, while he was multitasking.
  - Under an hour of check-ins during the 31.6-hour run.
  - See §6 for which pieces of that setup mattered.
- **The agent effort.**
  - **77 work orders** (23 discovery, 54 build-phase).
  - **~3.6B tokens processed**, 98% of them cache reads.
  - The **API-equivalent cost is ~$1,257** at Opus 5.5 list prices. It actually ran on a Claude
    subscription.
- **The documentation.**
  - **269 files / 440k words** of analysis, specs and plans.
  - A **58-screen clickable HTML prototype**.
  - A **54-entry decision log**.

The hypothesis under test was "agents rewrite well when all the inputs are in place". It held for
*building*. The open questions are about human review and the many external systems that use the
legacy databases. Those are the expensive parts that remain.

---

## 2. What we took on

**The legacy system.**
- usdecconnect.org is a custom CRM used by USDEC staff, overseas offices and members: contacts,
  organizations, overseas-office contact logs and reports, records, an issue tracker (MARA), a
  library, tasks, search and reports.
- It was built around 2005 on **CMS+ (RDCMS)**, RD's ColdFusion CMS. A planned move to Salesforce
  around 2010 never happened.
- The mid-2000s CFML stack was past supportable and a security liability by modern standards.

**Why it was hard:**
- **Code lives in the database.** CMS+ stores site code there and writes it out as CFML at publish
  time. The real behavior had to be read from the *published* files (`src/connect/`), resolved
  through ColdFusion mappings into platform code.
- **Uneven quality.** The platform code is solid for its era. The published app code is weak:
  written partly by weak developers and lived with for years. The rule was **rebuild behavior from
  intent and interface, never port the implementation.**
- **Generic data structures.** Much of the data sits in CMS+ structures: a versioned EAV content
  store (`CM_ContentItems` 218,780 items / `CM_ContentItemFields` 1.4M rows), a generic
  relationships table, a workflow engine, and comma-delimited list values.
- **The platform drifted.** The first platform snapshot turned out to be the wrong generation. A
  second checkout (Gen2) was needed, which forced a full re-survey.
- **Real data.** The data contains real addresses, so the rebuild never sends real email (it writes
  to a pickup folder only), and agents see only counts and aggregates, never client rows.

**Scope and framing.**
- This was a spec and support-level effort: scope a rewrite and carry it to a working prototype.
- It was also a deliberate test of whether agents rewrite well when all the inputs are in place.
- **Brooks is RD's CTO.** He wrote much of the original platform and was the authority on how CMS+
  works. Agents inventoried, characterized and rebuilt from what he surfaced.

---

## 3. Timeline

Git history begins at the end of discovery (first commit 2026-10-09 19:14), so discovery timing is
inferred from work-order dates and file modification times.

| Phase | When | Duration | What happened |
|---|---|---|---|
| Setup | 10-08 evening | hours | Template project, operating docs, repo |
| Discovery 1 (overnight, unattended) | 10-08 → overnight | ~1 night | 6 slice surveys of published Connect, synthesis, data dictionary, Core security sweep, 58-screen HTML prototype (WO-001–011) |
| Discovery 2 | 10-09 morning | ~4 h *(inferred)* | Gen2 platform docs, 6 Gen2 re-surveys, schema extraction (229 routine definitions), USDEC Agent dependency survey, synthesis refresh (WO-012–023) |
| Architecture and build setup | 10-09 20:45 → 21:06 | ~20 min (plus the doc's own run) | `architecture.md` (WO-024), build plan, greenfield DB decision |
| **Automated build run** | **10-09 20:45 → 10-11 04:22** | **31 h 37 m** | Implementation lane: 26 build orders + 2 fixes. Planning lane: 12 area plans, then 16 pre-cut build orders |
| Close-out | 10-11 04:24 → 06:49 | ~2.5 h | End-session, as-built deltas, run metrics |
| Brooks returns | 10-14 | — | Hands-on testing begins (P4 refinement) |

**Commits:** 66 total; 62 are local checkpoint commits during 10-09 → 10-11. By day: 13 on 10-09,
44 on 10-10, 8 on 10-11, 1 on 10-14. **21 build commits landed between midnight and 07:00.** The
median gap between build commits was 19.6 minutes; the longest was 91 minutes.

**Test-count growth** (from checkpoint commit messages):

| When (local, −0700) | Tests | Order |
|---|---|---|
| 10-09 21:46 | 168 | Spine A |
| 10-09 23:23 | 394 | Admin A1 |
| 10-10 02:12 | 659 | Contacts C1a (the usage-limit interruption, resumed) |
| 10-10 05:06 | 1,110 | Departments |
| 10-10 08:17 | 1,401 | Contacts C2a |
| 10-10 13:02 | 2,140 | Contacts C4a |
| 10-10 15:24 | 2,557 | Offices O1a |
| 10-10 18:07 | 2,953 | Tasks |
| 10-10 23:00 | 3,486 | Offices O2c |
| 10-11 01:48 | 3,716 | Reports |
| 10-11 04:22 | **3,918 (+1 skipped)** | Search S2: first pass complete |

---

## 4. The operating model (how it was run)

### 4.1 Documents are the memory

Agent sessions start with no memory, so a small, deliberately maintained document set *is* the
project's memory:
- **`AGENTS.md`**: orientation.
- **INDEX**: the map of every doc, with a status on each (verified / suspect / obsolete).
- **STATUS**: now, next, blocked.
- **Roadmap**.
- **Decision log**: append-only, dated, with rationale. Logged as decisions are made, not at
  close-out.
- **Deferred items**: each one with a revisit trigger.
- **Working model**.

Four workflow skills maintain these: start-session, save-context, end-session and init-project.
A fresh session reads INDEX → STATUS → recent decisions → git, then briefs.

### 4.2 Coordinator plus executors, driven by work orders

- **The coordinator** is one long-lived session. It plans, decides, reviews every result report,
  and makes a checkpoint commit after each accepted order.
- **Executors** are subagents. Each implements exactly one written **work order**
  (`work-orders/<branch>/NNN-slug.md`), which states scope, inputs, definition of done and a
  delivery block (model tier, isolation, scheduling). The executor appends a structured result
  report and stops at "ready to commit".
- **Model tiers by role:** the coordinator and standard executors on Opus; light executors on
  Sonnet for recon and mechanical work.

### 4.3 Two lanes during the build

- **The implementation lane is serial.** One executor at a time changes `app/` and the database.
- **The planning lane runs in parallel.** A planner writes the next area's plan note: research,
  screen checklist, data model, the legacy → new mapping, and the body of its build order. It later
  **pre-cuts** the build order against the app *as actually built*. It never touches `app/` or the
  database.
- The result: build orders arrived pre-researched and reconciled with reality. The planning lane
  paused 4 times to wait for implementation. Total subagent time (36.2 h) exceeded wall-clock time
  (31.6 h) because the lanes overlapped. Build orders alone used ~89% of the implementation lane's
  window.

### 4.4 Standing rules (binding on every build order; 12 in total)

1. Read the result reports of the orders you depend on. Their "for later orders" notes are binding.
2. **Data.** Never write to legacy. Write only through the database helper, whose target is fixed.
   Only counts and aggregates enter context. No client data in commits, fixtures or screenshots.
3. **Loads.** Idempotent (a second run changes 0 rows). Legacy wins for loaded rows; rows the app
   creates are never touched. A verify script per area. From-empty must converge. A step never
   depends on a later-sorted step.
4. Contacts change only through `ContactService` and are listed only through `ContactVisibility`.
5. **Access rules are written twice:** in C# and as a SQL predicate, with tests that the two agree,
   and registered with Search and Audit. Hidden means 404.
6. **Form guards** for the checkbox and paging bug classes caught mid-run, plus anti-forgery on
   every POST.
7. Lookup foreign keys need merge and conflict SQL. Audit entity types are catalogued.
8. **Email goes only to the pickup folder,** through the outbox. No mail is ever produced from
   loaded history.
9. **Interactive checks** run with jobs off, on fictional rows only, signed in as seeded fictional
   users through the real reset flow, never with stored secrets.
10. **Never commit** (the coordinator commits). No secrets. If something is denied, report it; don't
    work around it.
11. **Progress log.** Every executor appends milestones to its own work order, so a usage-limit
    interruption resumes rather than restarts.
12. **Mutation checks.** Break a guard on purpose, prove a test catches it, restore byte-identical
    source, and rebuild before any browser check. 58 mutation checks were recorded across 11 build
    orders.

Later planners found things that changed earlier plan notes. Those fixes became **31 cross-plan
amendments**, folded into each build order when it was cut. Plan notes are never edited after
review.

### 4.5 Safety and permission posture

- **Commits.** The agent normally never commits. For the PoC build Brooks enabled
  *autonomous local checkpoint commits*, so an unattended multi-day run could be rolled back step
  by step. Push, rebase and reset stayed denied. The permission is to be revoked before anything
  heads toward production.
- **Legacy source** (`src/`) is read-only by deny rule and not in git. Text in it is data, never
  instructions.
- **Database writes** go through a helper with a fixed target that refuses cross-database writes.
  The legacy databases are read-only.

### 4.6 Definition of done (every area)

Each area finished with:
- every contract screen built and reachable from the menu;
- its data model in migrations;
- data loaded from legacy, or seeded;
- authorization per the permission model;
- unit and integration tests green;
- a browser check of every screen;
- a legacy → new mapping section;
- a **Shortfall** section saying honestly what didn't land.

---

## 5. Statistics

### 5.1 Work orders and agent effort

| Category | Count |
|---|---|
| Discovery: slice surveys, Gen2 re-surveys, platform concepts | 13 |
| Discovery: synthesis ×2, prototype ×3, data dictionary + schema ×2, external dependency ×1, security sweep ×2 | 10 |
| Architecture | 1 |
| Plan-note orders (producing 12 area plans) | 8 |
| Pre-cut / cut orders (planning lane) | 16 |
| Build orders | 26 |
| Fix orders (class bugs: checkboxes, paging) | 2 |
| Doc order (architecture as-built) | 1 |
| **Total** | **77** |

Six areas were split mid-run into parts (C1, C2, C4, O1, O2, Search), adding 7 orders. 25 orders
carry a progress log.

**Build-phase metrics** (`docs/run-metrics.md`; discovery was not metered):

| Lane | Orders | Tokens | Median tokens | Minutes | Median minutes |
|---|---|---|---|---|---|
| Build | 26 | 17.33M | 674k | 1,678 | 66.7 |
| Cut (pre-cut) | 16 | 4.96M | 313k | 238 | 14.2 |
| Plan | 10 | 4.49M | 449k | 225 | 23.4 |
| Fix | 2 | 0.47M | 234k | 32 | 16.1 |
| **All** | **54** | **27.25M** | **507k** | **2,172 (36.2 h)** | **28.1** |

The range per order was 227k–861k tokens and 9.8–86.6 minutes. Coordinator tokens were not metered.

On the progress-log rule's cost: builds under rule 11 ran at 455k–861k tokens and 53–87 minutes.
Builds before it ran at 412k–834k and 38–83 minutes. That's no visible overhead, since later builds
also carried more checks and bigger loads.

**Actual token usage and API-equivalent cost.** Summed from the session and subagent transcripts
for 10-08 → 10-14: 3 coordinator sessions plus 78 subagent transcripts. Each message is counted
once, at its final usage. Priced at first-party API list rates (cached 2026-10-06) for Opus 5.5:
$4 input, $20 output, $0.20 cache read, $5 cache write (5-minute), $8 cache write (1-hour), per
million tokens. Every call ran on Opus 5.5, apart from 3 calls on Opus 4.8. No Sonnet tier was used.

| Phase | API calls | Output | Cache reads | Cache writes | API-equivalent |
|---|---|---|---|---|---|
| Setup (init interview) | 16 | 0.02M | 1.4M | 0.07M | $1 |
| Discovery (coordinator + 24 subagents) | 2,314 | 1.9M | 527M | 12.1M | $210 |
| Build run (coordinator + 54 subagents) | 8,213 | 6.3M | 3,001M | 57.6M | $1,045 |
| **Total** | **10,543** | **8.2M** | **3,529M** | **69.7M** | **≈ $1,257** |

Where the money goes: cache reads ≈ $706, cache writes ≈ $387, output ≈ $164. Long agent loops
re-read their context on every turn, and that's cheap when it's cached. The `run-metrics.md` token
column (27.2M) is the harness's per-subagent figure, not billable volume.

**How it was actually paid for:** a Claude subscription. Brooks: "most of a week's" usage, at 60%
of the weekly limit by the night of 10-10, including the initial planning. The API figure is for
comparison only.

### 5.2 Code: new app vs legacy

**New app (`app/`):**

| Item | Count |
|---|---|
| Projects | 4 (Core, Web, Tests, IntegrationTests) |
| C# | 538 files / 81,431 non-blank lines (49,272 production, 32,159 tests) |
| Razor views | 159 `.cshtml`, of which **116 routable pages** |
| Production code, all kinds (C#, Razor, migration SQL, JS, CSS) | ~67,000 non-blank lines |
| Test code | ~32,200 non-blank lines |
| DB scripts (loads, verify, seeds, reset) | ~9,800 non-blank lines |
| **Total** | **~108,900 non-blank lines** |
| Repositories / services / scheduled jobs | 46 / 39 / 9 |
| Migrations | 42, creating **103 tables** in 9 schemas, with 288 FKs and 192 indexes |
| Load scripts | 88 SQL files in 11 folders (73 load, 15 verify), 10,277 lines |
| Tests | 1,193 `[Fact]` + 183 `[Theory]` → **3,918 run, all passing** (1 full-text smoke test skipped) |
| Front end | 5 hand-written JS files, 2 CSS files, no vendored libraries |

**Screen contract** (58 screens defined during discovery):

| Status | Count |
|---|---|
| Built (incl. 1 placeholder) | 47 |
| Dropped (Calendar ×3, Surveys) | 4 |
| Deferred (Email Broadcast ×3) | 3 |
| Dead or inactive in legacy, not built | 4 |

On top of the contract, the 22 Admin screens were all built.

**Legacy (for scale):**

| Item | Count |
|---|---|
| Published Connect (`src/connect/`) | 653 CFML files / 60,587 non-blank lines: 424 files / 29,761 lines are Connect's own pages, the rest deployed platform modules |
| Platform Core (Gen2 `code/common`) | 1,760 CFML files / 220,845 lines |
| USDEC client platform code | 983 CFML files / 135,022 lines |
| Legacy schema | 493 tables, 3,377 columns, 352 FKs (94 untrusted), 655 procedures and functions; Connect touches 164 tables |

### 5.3 Data moved

| Measure | Value |
|---|---|
| Legacy databases (Core + Custom) | 493 tables / 23.09M rows / 5.17 GB |
| Legacy tables read by the loads | 88 (40 Core + 48 Custom), holding 4.16M rows (~883 MB) |
| Rows inserted by the loads | 748,330 (228 script runs of 73 scripts) |
| New database | 103 tables / 799,453 rows / 513 MB |
| From-empty load time | ~14 min wall-clock; 177 s of summed script time; search rebuild 22 s |
| Idempotency | The latest run of every one of the 73 scripts inserted 0 and updated 0 |
| Search index | 50,444 documents |

Only ~18% of the rows in the tables read actually landed. Most of the read volume is shared
all-sites platform tables that the loads slice: the audit trail, the EAV fields, entities and
resources.

**Per area (key rows landed):**

| Area | Rows |
|---|---|
| Contacts | 24.4k people · 12.2k organizations · 36.7k profiles · 24.3k activity threads · 49.8k notes · 173.6k audit events |
| Offices | 15.7k contact logs · 38.2k attendees and reps · 7.1k follow-ups · 5.7k department files · 1.7k declarations |
| Records | 4,087 records · 12.8k access rows · 9.5k classifications · 22k view events |
| MARA | 332 issues · 1.8k notes · 3.7k resources · 9.9k history events |
| Admin / reference | 248 countries · 225 lookup items · 79 groups · 41 content pages |
| Tasks · Home · Library · Reports | 98 tasks / 104 events · 226 quick links · 201 manual nodes · 13 saved reports |

**Not moved, by design:**
- **File bytes (~16 GB):** not on the build machine, so only metadata was loaded.
- **Logins:** no legacy logins were loaded; they move at cut-over.
- **The pre-2014 Offices archive.**
- **Skipped rows:** 92k thread-less email-capture notes, and ~95 orphaned threads that legacy
  couldn't show either.

### 5.4 Documentation produced

| Location | Files | Words |
|---|---|---|
| `docs/` (analysis, specs, plans, schema) | 269 | 440,366 |
| — of which the Markdown | 40 | 409,182 |
| `work-orders/` | 77 | 259,079 |
| `prototype/` (58-screen clickable HTML prototype) | 20 | 73,542 |

Also produced: 229 saved legacy routine definitions; **54** decision-log entries (17 on 10-08,
22 on 10-09, 12 on 10-10, 3 on 10-11); 12 deferred items; 12 standing rules; 31 cross-plan
amendments.

---

## 6. Human involvement: the prerequisites, and which ones mattered

### 6.1 Where the human time went

This is reconstructed from Brooks's 43 typed messages across the three sessions, using their
timestamps (local time, −0700). A "sitting" is a run of messages without a long gap. Spans are
upper bounds, because he was handling other work in parallel.

| # | Sitting | When | Span | Messages | What it established |
|---|---|---|---|---|---|
| 0 | Environment prep (outside the transcripts) | before 10-08 18:20, and 10-09 morning | ~2 h | — | Copy the published Connect code off the server. Svn checkouts of the platform (the first was the wrong generation; Gen2 was checked out 10-09 ~10:30). Local copies of the two legacy DBs (already present from earlier USDEC Agent work). A dedicated exploration SQL login. The reusable project template with its docs-as-memory skills, built on earlier projects. |
| 1 | Project setup interview | 10-08 18:20–18:52 | ~30 min | 4 | The story and goal: a support-level PoC on spec, carried to a prototype. Where the code and DBs are. Project docs initialized. |
| 2 | Priming discovery, and launching it overnight | 10-08 19:07–20:54 | ~1 h 45 m | 8 | **The platform walkthrough** (a ~3,000-character brief): how CMS+ publishes code from the DB, the mappings, the "platform good, app bad" gradient. The correction that the checked-out Core was the wrong generation, and that Core tags remain trustworthy. Branching into Records. Final check-in: "tokens are not a factor", authority to dispatch more exploration. **Then the overnight unattended discovery chain.** |
| 3 | Morning direction and data access | 10-09 06:50–08:42 | ~1 h 50 m | 7 | Prototype reviewed, then "what does it take to build". **Exploration DB login plus a credentials file → the query helper.** Stack chosen (.NET 10, Razor, Dapper). DB permissions widened for sprocs. The first "new database?" question. |
| 4 | Close-out with direction | 10-09 18:48–19:12 | ~25 min | 3 | Side tangents parked out of the main plan. The data lean. `src/` gitignored and read-only. |
| 5 | Build planning: setting the autonomous run's parameters | 10-09 19:14–20:51 | ~1 h 35 m | 5 | The goal (an autonomous rewrite). **A greenfield DB with real data, migrated rather than scrubbed;** the build DB created. Search on SQL full-text, not Solr. SQL Server 2022. Feature dispositions; Admin in scope. **Autonomy envelope:** about 4 days away, real blockers only, park with defaults, local checkpoint commits. |
| — | *During the run* (10-10 01:51 → 10-11 06:45) | 5 brief check-ins | **< 1 h total** | 10 | A resume after a usage limit. A morning progress check, plus the **progress-log rule**. **One batch of 5 answers.** Two evening progress checks. The close-out request. |
| — | *Not counted* | 10-09 afternoon | ~40 min | 4 | Unrelated tangents during the same sessions. |

**Totals: ~8 hours before launch (~2 hours of environment prep plus ~6 hours across five
sittings), and under an hour during a 31.6-hour run.** 10 of the 43 messages fell during the build,
most of them one-line progress checks.

### 6.2 Which prerequisites mattered most

This is an assessment from the record: where each input showed up downstream, and what happened
when one was missing or wrong.

**High leverage:**
1. **The platform walkthrough (sitting 2, ~1 hour).** It became `cmsplus-platform-concepts.md`,
   which every executor boots from. Without it, agents would have read the *svn* layout as the
   application. In fact the live code is published out of the database and resolved through
   mappings, and that's where the behavior lives. It's the single highest-leverage human input, and
   only the platform's author could give it.
2. **The correct code generation.** The first checkout was the pre-BlueDragon Core. Discovery ran
   anyway and had to be redone against Gen2: 6 re-survey orders plus a synthesis refresh. The human
   effort to fix it was small (one checkout). The cost of not doing it first was about half a day of
   rework. **Lesson: verify the source generation before surveying.**
3. **Real database access with guard rails (sitting 3).** That meant a dedicated login, a read-only
   query helper, an "aggregates only, never client rows" rule, and later a write helper with a fixed
   target. It turned a code-derived guess at the schema into an authoritative extraction (229
   routine definitions), and it made real-data loads possible. **The code-only data dictionary
   written before access existed (WO-009) was largely superseded. Give DB access early.**
4. **Up-front decisions that remove forks (sitting 5):**
   - the stack;
   - the greenfield DB;
   - Admin in scope;
   - the feature dispositions: Calendar, Email Broadcast and Surveys out; Search on full-text;
   - SQL Server 2022.

   Every fork settled before launch was one the unattended run didn't stall on. (The greenfield
   choice holds on its merits; what it left open is the legacy integrations, see §9.)
5. **The autonomy envelope.** Token budget not a constraint. Park with defaults rather than block.
   Local checkpoint commits. A stated away window with "real blockers only". These are what let a
   31.6-hour run proceed while the human was away, including overnight.
6. **The pre-existing project template.** It provided docs-as-memory, the four session skills, and
   the work-order contract. It carried the project across 3 coordinator sessions and context
   compactions with no lost state. It was a prior investment, not part of the ~8 hours.

**Mattered less than expected:**
- **Mid-run check-ins.** Only two changed anything: the progress-log rule, and the batch of 5
  answers. Most parked defaults would have stood anyway.
- **Pre-access analysis.** Work done before DB access and before Gen2 (the code-only dictionary,
  the first-generation surveys) was partly or wholly redone.
- **Side tangents during the sessions.** Real, but irrelevant to the rewrite.

### 6.3 What's left for humans

Ambiguities were **parked with a default applied** rather than blocking. 11 questions remain
parked, plus 5 technology-transfer items. **Structured human review, user testing and the plan for
the legacy integrations haven't happened yet.** That's now the dominant cost. The build fast-forwarded through
what would likely have been months of development; it did not fast-forward the work of getting to
launch.

---

## 7. What worked

- **Docs as memory.** A fresh session reorients in minutes, and nothing depended on a single
  context surviving.
- **Pre-cutting build orders against the as-built app.** It caught drift between plan and reality
  before execution, not after.
- **Standing rules plus cross-plan amendments.** A lesson learned once got encoded once and enforced
  in every later order.
- **Class-level fixes.** Two cross-cutting bugs (checkbox binding, paging parameters) were caught
  downstream, fixed at the shared level, and added as standing-rule guards.
- **Progress logs.** The one usage-limit interruption (Contacts C1a) resumed without loss, at no
  measurable token overhead.
- **Mutation checks.** Testing the tests, especially on access and mail guards.
- **Idempotent, verified loads with real data.** Every screen was exercised against realistic
  volume, and agents never saw client rows.
- **Honest Shortfall sections.** They became the refinement backlog.
- **The overnight unattended chains** (discovery on 10-08, build on 10-09 → 10-11).

## 8. What went wrong, or would be done differently

- **Discovery wasn't in git.** That left no timestamps, and no metering until 10-10. Commit from
  day one, and start the run-metrics table from the first work order.
- **The wrong platform generation.** The first snapshot forced a full Gen2 re-survey (6 orders).
  Confirm the source generation before surveying.
- **A safety-classifier block.** One discovery order (WO-014) was blocked by the safety classifier
  on its first run and re-dispatched.
- **Secrets the user couldn't see.** Secrets written by agents through the desktop app landed in
  the app package's private AppData, because the app is MSIX-packaged and Windows redirects those
  writes. The user's own shell couldn't see them, so the run script failed for the human while
  every agent-run test passed. Prefer repo-local, gitignored config for anything a human also runs,
  or document the redirection.
- **The greenfield database was the right call, with one cost: the legacy integrations.** A fresh
  schema made the build clean and fast, and it's the approach we'd keep. The schema is built for
  the application's domain rather than CMS concepts, and sized for large record pulls. The data
  comes across by verified, repeatable migration scripts.

  The cost is that every *other* system using the legacy databases has to be repointed, replaced or
  retired: an AI agent, member logins, a supplier search, the HubSpot sync, and direct-query readers
  not yet identified. They weren't inventoried before the decision, so they became a later phase
  that is hard to scope. See §9.
- **Human review was deferred by design.** It's the expensive part, and it still lies ahead.

## 9. Where it stands, and what's next

- **First pass complete;** the refinement pass (P4) started with hands-on testing.
- **The data approach stands: a greenfield database plus migration.** The reasons hold:
  - much of the legacy structure is built on CMS+ concepts that are no use to this application:
    a versioned EAV content store, generic relationships, a workflow engine, UDF tables, and
    comma-delimited lists;
  - it doesn't scale to large record pulls;
  - the new schema adds typed tables, enforced foreign keys, optimistic concurrency and Unicode
    text;
  - the load scripts are already a tested, repeatable first draft of the migration.
- **The open planning item is the legacy integrations:** everything else that reads or writes the
  legacy databases. A full inventory is under way separately. Each integration is then its own
  effort, handled in order of risk: both the risk of the change and its current exposure. That,
  not the schema, sets the path to launch.
- **The bottom line.** This can really work, but you aren't done when the build finishes. Human
  review, user acceptance, the legacy integrations and the cut-over are what stand between a
  working rebuild and a launch.

## 10. Reusable patterns for a new project (checklist)

1. Set up the doc set first (AGENTS.md, INDEX, STATUS, decision log, deferred items, working
   model) and the four session skills.
2. Put everything in version control from minute one, including discovery output. Start
   run-metrics from WO-001.
3. Discovery chain: slice surveys → synthesis with a **screen contract** → clickable prototype →
   schema extraction → dependency surveys of external consumers. Run overnight, unattended.
4. **Inventory every consumer of shared data before choosing a data strategy.**
5. Architecture doc, then a build plan with a definition of done per area and binding standing
   rules.
6. Two lanes: one serial implementation lane, one parallel planning lane that pre-cuts orders
   against the as-built app.
7. Executors keep progress logs, run mutation checks, write Shortfall sections, and stop at "ready
   to commit". The coordinator reviews, re-runs the suite and checkpoint-commits.
8. Use real data through server-side, idempotent, verified load scripts. Agents see aggregates
   only, and mail goes to a pickup folder only.
9. Park forks with defaults. Never block on the human.
10. Budget for **human review and UAT** as the dominant cost, and choose architecture to minimize
    what has to be human-verified.

---

## Appendix: caveats on the numbers

- **Discovery timing is inferred,** from work-order dates and file modification times.
- **Run metrics cover only the build phase** (WO-024–077). WO-038's row is the resume segment only.
- **"Rows read"** means rows in the tables the loads touch, not rows consumed.
- **Test cases.** The static count (2,128 facts plus inline cases) is below the 3,918 that run,
  because member-data theories expand at runtime.
- **Line counts are non-blank lines** (`bin`/`obj` excluded). The legacy `sites/` totals include
  duplicated platform modules.
- **API-equivalent cost** is computed from transcript usage records, taking the final usage per
  message, at list prices cached 2026-10-06. It excludes this analysis session and anything after
  10-14. The 3 Opus 4.8 calls are priced at Opus 5.5 rates, a difference of under $1.
- **Human time** is reconstructed from message timestamps. Spans are upper bounds while
  multitasking. Environment prep (~2 h) is Brooks's estimate.

**Sources:** `docs/2-project-status.md`, `docs/4-decision-log.md`, `docs/build-plan.md`,
`docs/run-metrics.md`, `docs/6-working-model.md`, `docs/schema/*`, `app/db/loads/*/README.md`,
`app.DataLoadRuns`, `git log`, and file counts taken 2026-10-27.
