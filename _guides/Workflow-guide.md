---
type: guide
tags: [sandbox]
---

# Workflow-guide.md

> How the Lab is used day-to-day: capture → sort → do → review. Flows for each role.

The Lab has one core loop and four role-specific branches. Everything routes back to [[Home]].

---

## The core loop (every day)

```
Ctrl+Shift+Space  →  Inbox.md  →  Tasks.md  →  [workspace dashboard]  →  Home.md
```

1. **Capture** — use `Ctrl/Cmd+Shift+Space` from anywhere. The modal asks for text + kind (thought/task/link). It writes to today's file in `Inbox/YYYY-MM-DD.md` via `H.captureToInbox()`. No thinking required.

2. **Sort** — open [[Inbox]], select today's day in the day selector. Scan entries:
   - **Task items** `- [ ] ...`: leave them, or open them inline from the [[Tasks]] dashboard.
   - **Thoughts**: promote to a note if actionable, or leave in Inbox for the Weekly Review.
   - **Links**: note the context (what you were reading/doing when you saved it).

3. **Do** — navigate via the rail to the workspace that matches what you're working on:
   - Studying → [[Academics]] or [[Programming]]
   - Building → [[Projects]]
   - Job hunting → [[Jobs]]
   - Research → [[Research]]
   - Maintenance → [[Life]]
   - Reading → [[Library]]

4. **Return** — the floating Home button (bottom-left, every dashboard) brings you back. Check if anything new surfaced.

---

## Role flows

### Student flow

```
Home → Academics → Programming → Tasks → Lo-fi Workspace
```

**Morning**
- [[Academics]]: check assignment due dates, course progress
- [[Programming]]: update a concept confidence dot (1–5), log which algorithm you reviewed

**Study session**
- [[Lo-fi Workspace]]: set the session label (e.g. "Algorithms"), tie to a project/course if you want the log tracked. Start the 25-minute timer. The session writes to `Sessions/YYYY-MM-DD.md` when complete.

**Post-session**
- [[Tasks]]: tick off any checkboxes you completed during the session
- Open the concept note, update `confidence` if your understanding changed (Programmig dashboard does this inline; or edit frontmatter directly)

---

### Job hunter flow

```
Home → Jobs → Applications/ → Tasks
```

**Job search**
- From the Create menu (nav rail **+**): choose "Job application" → fill in company, role, status = `saved`
- The light template auto-applies when the file appears in `Applications/`

**Application progression**
- [[Jobs]]: change the stage dropdown inline (Preparing → Applied → Recruiter screen → Interviewing → …)
- Each status maps to a value in `STATUS.job`; only pipeline-visible stages show in the funnel view

**Interview prep**
- From Create menu: choose "Interview", link to the application
- Fill in `Interview.md` template: before-during-after sections, questions to ask, reflection

---

### Reader flow

```
Home → Library → Books/ → Tasks
```

- [[Library]]: browse current reads with progress bars and ratings
- Use the Open Library search to add new books (title/author/ISBN, one-click import with cover art)
- Set a reading goal via "Set goal" — persists in sandbox storage
- Book note lives in `Books/`; update `progress`, `current_page`, and `rating` from the note or via the dashboard

---

### Builder flow

```
Home → Projects → Tasks → Playground
```

- [[Projects]]: click "New project space" → names the project, creates `Projects/<Name>/` with `Notes`, `Research`, `Assets`, `Changelog.md`
- Work inside the project folder; tasks in its notes surface automatically in [[Tasks]]
- Status changes on the project card update the source note's frontmatter via `patchFrontmatter`
- [[Playground]]: come here when you want to re-encounter old projects or ideas — "Show me something" surfaces a random Lab page

---

## Review flows

### Weekly (Friday)

Open **Weekly Review** (via Create menu or `_templates/Weekly Review.md`). The template includes a live dataviewjs table pulling this week's:
- Study/coding hours, pages read, energy/focus signals
- Tasks completed this week (from your Inbox/notes)

Then manually:
1. Process remaining Inbox entries: promote, archive, or delete
2. Check [[Tasks]] for stale items (due date in the past) — reschedule or complete
3. Update [[Projects]] if any status changed this week

### Monthly

Open **Monthly Review** (`_templates/Monthly Review.md`). 30-day aggregate signals, then:
1. What compounded (projects that moved, courses that progressed)
2. What to release (archived projects, abandoned reads)
3. Next month's focus area

---

## Status change flow (where it surfaces)

Every status update in the Lab writes back to the source note's frontmatter. Here's where each change happens and where it shows up:

| Status | Where you change it | What it affects |
|---|---|---|
| Project status | [[Projects]] inline dropdown | `ACTIVE_PROJECT` filter, Home active-projects count, Vault Health unknown-status check |
| Job stage | [[Jobs]] inline dropdown | Career funnel visibility, closed stages drop out of pipeline |
| Book status | `Books/<name>.md` frontmatter or [[Library]] | Library sorting, reading-list display |
| Task completion | Click any checkbox (any dashboard) | Source-line mutation; disappears from [[Tasks]]; surfaces in Weekly Review "done" list |
| Concept confidence | [[Programming]] radar dots | Persists to `confidence` frontmatter field |
| Assignment status | `Assignments/<name>.md` frontmatter | Visible in [[Academics]] assignment list |

---

## Quick commands

| Shortcut | What happens |
|---|---|
| `Ctrl/Cmd+Shift+Space` | Quick capture to today's Inbox (thought/task/link) |
| Nav rail **+** | Typed note creation (form modal with title, tags, related) |
| Nav rail icon click (phone) | Toggle rail open/closed (draggable on mobile) |
| Home button (any dashboard) | Navigate to [[Home]] |
| Vault Health link (rail) | Diagnostics — run when something feels off |