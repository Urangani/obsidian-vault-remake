---
type: guide
tags: [sandbox]
---

# Template-guide.md

> How note templates work in the Lab: the two tiers, the auto-skeleton engine, and exactly what to touch when you add a new domain.

---

## Two tiers

| Tier | Location | When used | Content |
|---|---|---|---|
| **Light** | `_templates/light/` | Auto-applied to **empty** `.md` files created/moved into a mapped domain folder | Frontmatter only + title heading — never clobbers content |
| **Full** | `_templates/` | Explicit creation via `H.createNote()`, the nav-rail **+** menu, or Templater | Structured sections, callouts, task lists, placeholders |

The **skeleton engine** (Atelier Tools plugin) applies light templates. The **Create menu** (`atelier-tools:new-note`) applies full templates.

---

## Light templates (13)

Frontmatter-only. These exist so a brand-new note in a domain folder is *queryable* immediately (shows in dashboards, gets typed) without any human effort.

| Domain folder | Light template | Auto-added fields |
|---|---|---|
| `Books/` | `Book.md` | type, status (`reading`…), title, author, ISBN, rating, progress, pages, cover |
| `Projects/` | `Project.md` | type, status, area, priority, progress, dates, next_milestone, repository |
| `Courses/` | `Course.md` | type, status, term, credits, progress, next_assessment |
| `Assignments/` | `Assignment.md` | type, status, course, kind, weight |
| `Concepts/` | `Concept.md` | type, domain, status, difficulty, confidence, last_reviewed |
| `Algorithms/` | `Algorithm.md` | type, difficulty, confidence, last_reviewed, next_review |
| `Questions/` | `Research Question.md` | type, status, started |
| `Sources/` | `Source.md` | type, kind, url, accessed |
| `Areas/` | `Life Area.md` | type, status, focus, review |
| `People/` | `Person.md` | type, relationship, organization, contacts, context |
| `Works/` | `Work.md` | type, kind, creator, status, rating |
| `Applications/` | `Job Application.md` | type, company, role, status, location, work_mode, url, salary, priority |
| `Interviews/` | `Interview.md` | type, company, role, date, interview_type, interviewers |

## Full templates (21)

These live in `_templates/`. They carry the structure that makes notes worth opening:

- **Project.md** — Definition of done, why this matters, milestones, next actions, decisions
- **Book.md** — reading progress, notes by chapter, quotes, key ideas, review
- **Concept.md** — one-liner, smallest example, common mistakes, test yourself
- **Job Application.md** — role snapshot, alignment matrix, prep checklist, follow-up
- **Interview.md** — before/during/after, questions to ask
- Plus `Course, Assignment, Algorithm, Coding Problem, Code Snippet, Source, Research Question, Person, Work, Life Area, Focus Session, Study Session, Learning Path, Daily, Weekly Review, Monthly Review`

A full template commonly includes:
- `type:` + domain fields in frontmatter (copied from the light version, often richer)
- A title using `<% tp.file.title %>` or `{{title}}`
- Obsidian callouts (`> [!note]`) and checkbox lists (`- [ ]`)
- Dataview/DataviewJS blocks where live data is wanted (e.g. Weekly Review's signals table)

---

## The auto-skeleton engine

**Where:** `.obsidian/plugins/atelier-tools/main.js` → `setupSkeletonEngine()` (`main.js:183-219`).

**How it fires:**
1. Obsidian fires `create` or `rename` on a `.md` file
2. The engine checks the file's parent folder against `SKELETONS` (`main.js:18-24`)
3. If matched **and** the file is empty (`read().trim().length === 0`) **and** not `staging-` prefixed **and** not already in-flight, it writes the light template

**Why it's safe:**
- **Never touches non-empty files** — your content always wins
- `staging-` exclusion → `H.createNote()` can place a full-template note via `_core/staging-….md` then rename it into the folder without the skeleton clobbering it
- **`inFlight` set with a 1.5 s cooldown** makes create+rename double-firing idempotent

---

## Placeholders

Both tiers use the same placeholder syntax, replaced at creation time (by the skeleton engine, by `H.createNote()`/`createNote()`, or by Atelier Tools — **not** by Templater at runtime):

| Placeholder | Replaced with |
|---|---|
| `<% tp.file.title %>` | the note's filename (basename) |
| `<% tp.date.now("FORMAT") %>` | today's date in the given format |
| `{{title}}` | the note's basename |
| `{{date:YYYY-MM-DD}}` | today's date |

---

## Adding a new domain folder — 3 places to update

Add the folder and its light template in **all three** locations:

1. **Light template** → `_templates/light/<Kind>.md` (frontmatter-only)
2. **Auto-skeleton map** → `SKELETONS` in `.obsidian/plugins/atelier-tools/main.js` (`main.js:18-24`)
3. **Templater folder mapping** → `folder_templates` in `.obsidian/plugins/templater-obsidian/data.json` (`data.json:11-64`)

Optional (only if the domain should appear in the **Create menu**):
4. **Create menu** → the `T` helper entries in `labTypesFor()` in `atelier-tools/main.js` (`main.js:40-55`)
5. **Nav rail** → `NAV` in `_core/helpers.js` (`helpers.js:305-318`)
6. **`KIND_FRONTMATTER`** → `_core/helpers.js` (`helpers.js:36-52`) so a missed template still yields typed frontmatter

---

## Known gaps

Not every full template has a light twin. If the skeleton fires for one of these types, creation falls back to `KIND_FRONTMATTER` synthesis (still typed, but minimal):

| Type | Full template | Light template |
|---|---|---|
| `coding_problem` | ✅ `Coding Problem.md` | ❌ missing |
| `code_snippet` | ✅ `Code Snippet.md` | ❌ missing |
| `learning_path` | ✅ `Learning Path.md` | ❌ missing |
| `focus_session` | ✅ `Focus Session.md` | ❌ missing |
| `study_session` | ✅ `Study Session.md` | ❌ missing |
| `daily` | ✅ `Daily.md` | ❌ missing |
| `weekly_review` | ✅ `Weekly Review.md` | ❌ missing |
| `monthly_review` | ✅ `Monthly Review.md` | ❌ missing |

These are mostly non-folder types (reviews, sessions, daily notes) so the gap is low-impact — but if you ever create a mapped domain for them, add the light template. See [[Improvement-plan]].