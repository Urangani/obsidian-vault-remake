---
type: guide
tags: [sandbox]
---

# Structure-guide.md

> The folder map, frontmatter conventions, and status vocabularies that keep every note queryable.

---

## Folder map

```
vault root/
├── _core/                          ← shared runtime (helpers.js) + vendor libs
├── _templates/                     ← full templates (explicit creation)
│   └── light/                      ← light templates (auto-skeleton)
├── _guides/                        ← this documentation
│
├── (14 dashboard .md files)        ← Home, Inbox, Tasks, Academics, Programming,
│                                      Projects, Library, Jobs, Research, Life,
│                                      Culture, Playground, Lo-fi Workspace,
│                                      Vault Health
│
├── Inbox/                          ← daily captures (YYYY-MM-DD.md)
├── Sessions/                       ← focus session logs (YYYY-MM-DD.md)
│
├── Projects/                       ← type: project
├── Courses/                        ← type: course
├── Assignments/                    ← type: assignment
├── Concepts/                       ← type: concept
├── Algorithms/                     ← type: algorithm
├── Books/                          ← type: book
├── Applications/                   ← type: job_application
├── Interviews/                     ← type: interview
├── Questions/                      ← type: research_question
├── Sources/                        ← type: source
├── Areas/                          ← type: life_area
├── People/                         ← type: person
├── Works/                          ← type: work
│
├── Attachments/                    ← images, files
├── Notes/                          ← general notes (utility, reviews)
│
├── Home.md, Tasks.md, ...          ← dashboard root files
├── README.md                       ← vault description (external-facing)
├── START HERE.md                   ← first-open quickstart
└── AGENTS.md                       ← AI/developer architecture reference
```

---

## Frontmatter conventions

Every note must have a `type` field. Dashboard queries filter by `type`. The `type` value is always `snake_case`, singular.

### Required by type

| Type | Required fields | Common optional fields |
|---|---|---|
| `project` | `status`, `area` | `progress`, `priority`, `start_date`, `target_date`, `next_milestone` |
| `course` | `status`, `term` | `credits`, `progress`, `next_assessment` |
| `assignment` | `status`, `course` | `kind`, `weight`, `due_date` |
| `concept` | `domain`, `status` | `difficulty`, `confidence`, `last_reviewed` |
| `algorithm` | — | `difficulty`, `confidence`, `last_reviewed`, `next_review` |
| `book` | `status`, `title`, `author` | `ISBN`, `pages`, `rating`, `progress`, `current_page`, `cover` |
| `job_application` | `status`, `company`, `role` | `location`, `work_mode`, `job_type`, `salary`, `resume`, `priority` |
| `interview` | `company`, `role`, `date` | `interview_type`, `interviewers` |
| `research_question` | `status` | `started` |
| `source` | `kind` | `url`, `accessed` |
| `life_area` | `status` | `focus`, `review` |
| `person` | — | `relationship`, `organization`, `contacts`, `context` |
| `work` | `kind`, `creator`, `status` | `rating`, `date_encountered` |
| `dashboard` | — | — (cssclasses required instead) |

### Dashboards special fields

Every dashboard must have:
```yaml
type: dashboard
tags: [sandbox]
cssclasses: [dashboard-layout, atelier-dashboard-page]
```

The only exception is [[Lo-fi Workspace]], which uses `cssclasses: [atelier-lofi-page]` instead of the dashboard grid classes.

---

## Canonical status vocabularies

Defined in `_core/helpers.js:25-33`. Dashboard queries filter on these exact values — never invent a new status without adding it here first.

### Project statuses
```yaml
status: idea | planned | active | blocked | paused | complete | archived
```
`ACTIVE_PROJECT = ["idea", "planned", "active"]` — shown as "in motion" on Home and Projects.

### Job application statuses
```yaml
status: saved | preparing | applied | recruiter_screen | interviewing | offer | rejected | withdrawn | archived
```
`rejected`, `withdrawn`, `archived` are considered closed and filtered from the Jobs funnel.

### Book statuses
```yaml
status: want_to_read | reading | finished | paused | abandoned | revisit
```

---

## The `KIND_FRONTMATTER` fallback

When `H.createNote()` or the auto-skeleton can't find a matching template (full or light), it synthesizes frontmatter from `KIND_FRONTMATTER` (`helpers.js:36-52`). Notes are never left untyped. The synthesized frontmatter is minimal — just enough for the note to appear in dashboard queries.

If you want richer structure, use a full template or fill in fields manually after creation.

---

## Path resolution

The Lab works in two modes: standalone (root = vault root) or nested inside a parent vault (`Atelier Lab/`). Path resolution is handled by `H.path(rel)` — never hardcode `Atelier Lab/` or assume a root.

`H.q(folder)` returns a Dataview-safe quoted path for queries:
```js
dv.pages(H.q("Projects"))  // → dv.pages('"Projects"') or dv.pages('"Atelier Lab/Projects"')
```

`H.isLab(p)` returns `true` for any file within the Lab, regardless of nesting depth. Use it in filters to exclude templates, attachments, and other vault internals.

---

## Inlinks and orphans

Vault Health marks a note as an **orphan** if both `file.inlinks.length === 0` and `file.outlinks.length === 0`. Keep notes connected with `[[wikilinks]]` — even a single link to a related note or parent dashboard is enough.

The `related` field in frontmatter (populated by the Create modal) is a convention, not a query target. Wikilinks in the body text are what Vault Health checks.