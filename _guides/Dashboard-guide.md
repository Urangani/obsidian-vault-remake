---
type: guide
tags: [sandbox]
cssclasses: [atelier-dashboard-page]
---

# Dashboard-guide.md

> The map of every dashboard in the Lab: what each one does, how to read it, and where its data comes from.

Every dashboard is a markdown note with `type: dashboard` in the frontmatter and a single `dataviewjs` block. Each one loads the shared runtime (`_core/helpers.js`), mounts the floating Home button and the navigation rail, then renders a hero, signal panels, and domain content.

## Dashboards at a glance

| Dashboard | Tagline | Domain | Key data source |
|---|---|---|---|
| [[Home]] | "Good morning/afternoon/evening" | Whole vault | All `labPages()` with open tasks, active projects, reading books, jobs in progress |
| [[Inbox]] | "Capture" | Unexplored thought | `Inbox/YYYY-MM-DD.md` per-day captures |
| [[Tasks]] | "Everything in flight." | Task pool | Every open `[ ]` checkbox in the vault (`labPages().file.tasks`) |
| [[Academics]] | "Understand deeply." | Courses / assignments / concepts | `Courses/`, `Assignments/`, `Concepts/` |
| [[Programming]] | "Build fluency." | Concepts / algorithms / builds | `Concepts/`, `Algorithms/`, `Projects/` |
| [[Projects]] | "Ship small things." | Projects | `Projects/` + open tasks |
| [[Library]] | "Read" | Books | `Books/` + Open Library search |
| [[Jobs]] | "Build your next chapter." | Job applications / interviews | `Applications/`, `Interviews/` |
| [[Research]] | "Follow the question." | Questions / sources | `Questions/`, `Sources/` |
| [[Life]] | "Live deliberately." | Life areas | `Areas/` |
| [[Culture]] | "Pay attention." | Cultural works | `Works/` |
| [[Playground]] | "Make room for wonder." | Whole vault (serendipity) | Random `labPages()` |
| [[Lo-fi Workspace]] | — (immersive scene) | Focus sessions | `Sessions/YYYY-MM-DD.md` logging |
| [[Vault Health]] | "Keep the vault trustworthy." | Metadata integrity | All `labPages()` diagnostics |

## Common anatomy

All dashboards share the same skeleton. Once you can read one, you can read them all:

1. **Hero** — the date eyebrow (e.g. "PROJECT STUDIO"), a tagline, and a short "how to use this page" hint. The **orbit ring** (top-right) is a single headline signal: project count, open questions, books tracked, works count.
2. **Tab bar** — filter pills (e.g. project status, pipeline stage). Clicking re-renders the panel below.
3. **Main panel** — the working surface: cards, list rows, progress meters.
4. **Side panel** — "Signal" stats or secondary lists (e.g. open tasks, health metrics).
5. **Inline actions** — status dropdowns, confidence dots, task checkboxes, capture inputs. Many write straight back to the source note's frontmatter or markdown.

## Dashboard by dashboard

### [[Home]] — command center
- **Purpose:** The landing page. Daily orientation: who you are, what today's focus is, what's on the runway.
- **To read:** Hero greeting + your name (both clickable/editable), the day's focus line, open tasks, active projects, currently-reading books, active job applications.
- **Data source:** `H.labPages()` filtered to open tasks and `ACTIVE_PROJECT`, `Books/` with `status: reading`, `Applications/` not in closed stages. Focus text and name live in sandbox storage (`store`).

### [[Inbox]] — daily capture
- **Purpose:** Everything you grab during the day, grouped by day.
- **To read:** Day selector (persists), note/task toggle, inline capture input, list of that day's entries with inline completion.
- **Data source:** `Inbox/YYYY-MM-DD.md`. Writes via `H.captureToInbox()` or the `Ctrl/Cmd+Shift+Space` quick capture modal in Atelier Tools.

### [[Tasks]] — everything in flight
- **Purpose:** Single place to see and complete every open checkbox across the Lab.
- **To read:** All open tasks sorted by due date (nearest first), filterable by source-note type. Orbit ring = open task count. Weekly completion stats panel.
- **Data source:** `H.labPages().file.tasks` filtered to unchecked rows, excluding `_templates/`. Completion rewrites the source line (`helpers.js:78-89`).

### [[Academics]] — academic studio
- **Purpose:** Your courses, their assignments, and the concepts you're studying this term.
- **To read:** Active course cards with progress bars, assignments with due dates and weights, concept list. Orbit ring = active course count.
- **Data source:** `Courses/`, `Assignments/`, `Concepts/`; assignment rows join to their parent course by frontmatter `course`.

### [[Programming]] — programming lab
- **Purpose:** Track concept confidence over time, review algorithms, and see your build projects.
- **To read:** Tabs for **Radar** (confidence dots — click a dot to set 1-5, persisted to frontmatter via `patchFrontmatter`), **Algorithms**, and **Builds**.
- **Data source:** `Concepts/` (uses `confidence`, `domain`), `Algorithms/`, `Projects/`.

### [[Projects]] — project atelier
- **Purpose:** Every project with an inline-editable status pill. "New project space" creates a stamped hub folder (`Projects/<Name>/` with `Notes`, `Research`, `Assets`, `Changelog.md`).
- **To read:** Filter tabs (All / Active / blocked / paused / archived / Complete). Each row: index, title, next milestone (or "Next milestone undefined"), progress meter, status select. Clicking a row opens the note.
- **Data source:** `Projects/` where `type === "project"`. Status writes via `patchFrontmatter`. `ACTIVE_PROJECT = [idea, planned, active]` counts as "in motion."

### [[Library]] — library
- **Purpose:** Book tracking with cover art, page-level progress, ratings, and Open Library importing.
- **To read:** Book cards sorted by status; "Set goal" for reading target; search box for title/author/ISBN with one-click import.
- **Data source:** `Books/`. Covers resolve via vault API or Open Library; import writes a full frontmatter scaffold.

### [[Jobs]] — career studio
- **Purpose:** Applications funnel and interview room.
- **To read:** Pipeline stage dropdown per application (inline), tab bar for pipeline vs interviews. Closed stages (rejected/withdrawn/archived) are filtered from the funnel. Orbit ring = active thread count.
- **Data source:** `Applications/` (`job_application`), `Interviews/`.

### [[Research]] — research studio
- **Purpose:** Questions as first-class citizens, with their sources and related tasks.
- **To read:** Grid layout: open research questions (main), sources and tasks (sidebar). Orbit ring = open question count.
- **Data source:** `Questions/` (`research_question`), `Sources/`, plus tasks linked to them.

### [[Life]] — life studio
- **Purpose:** Deliberate maintenance of the areas that keep the machine running.
- **To read:** Life-area cards (health, money, home, quiet systems…) filterable, each with its tasks.
- **Data source:** `Areas/` (`life_area`).

### [[Culture]] — culture studio
- **Purpose:** Tracking what you paid attention to — films, music, essays, paintings.
- **To read:** Gallery of recent works; orbit ring = works-tracked count.
- **Data source:** `Works/` (`work`).

### [[Playground]] — atelier playground
- **Purpose:** Serendipity. "Show me something" surfaces a random Lab page; good for re-encountering old notes. Production vault unaffected.
- **Data source:** Random index into `H.labPages()`.

### [[Lo-fi Workspace]] — immersive focus
- **Purpose:** A full-screen three.js rain/dust scene with a persisted focus timer and a music playlist.
- **To read:** Start/stop timer (default 25 min), session label, project/course tie-in, playlist. Completed sessions append to `Sessions/YYYY-MM-DD.md` as checked focus-task lines.
- **Data source:** three.js vendor lib + `lofi-city.svg` background; sessions via `H.logFocusSession()`. Uses the `atelier-lofi-page` cssclass rather than a dashboard grid.

### [[Vault Health]] — trust layer
- **Purpose:** Find metadata drift: notes missing `type`, `progress` out of 0–100, statuses outside the canonical vocabularies, and orphan notes (zero links either way).
- **To read:** Health signal panel (lab notes / missing type / bad values / orphans) + two lists: "Notes needing attention" and "Isolated notes."
- **Data source:** All Lab pages except `_`-prefixed internals. Scoped via `H.isLab()`.

## The navigation rail

The left rail (see `_core/helpers.js:305-318`) is the cross-domain spine: Home, Inbox, Tasks, Academics, Programming, Projects, Library, Jobs, Research, Life, Culture, Vault Health, plus a **+** create button that opens the typed Create modal. On phones the rail becomes a draggable bubble (position persists per-device).

> People were removed from the rail in the audit: `People.md` was deleted and the Home tile dropped. `People/` and its templates still exist if a People studio is ever wanted.

## Editing a dashboard

1. Open the `.md` file in source mode.
2. Edit the `dataviewjs` block. Helpers come from `H`: `H.icon`, `H.open`, `H.sectionHead`, `H.empty`, `H.tabBar`, `H.taskRow`, `H.store`, `H.patchFrontmatter`, `H.labPages`, `H.q`, `H.path`…
3. Reload Obsidian (no hot reload) to see CSS/helper changes.