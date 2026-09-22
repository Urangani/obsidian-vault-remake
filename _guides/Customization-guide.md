---
type: guide
tags: [sandbox]
---

# Customization-guide.md

> How to change how the Lab looks and behaves: themes, CSS, the shared runtime, the nav rail, and vendored dependencies.

---

## Before you touch anything

Two invariants — respect them or dashboards will look broken:

1. **Two CSS snippets must be enabled** under Settings → Appearance → CSS snippets:
   - **`atelier`** — dashboard grid, panels, glass bento `adx-*` classes, nav rail
   - **`lofi-scene`** — rain overlay, timer styling, lo-fi workspace layout
2. **No hot reload.** Changes to `_core/helpers.js` or CSS snippets require an Obsidian reload to take effect.

---

## CSS classes roadmap

| Class | Used by |
|---|---|
| `adx` | Every dashboard root container |
| `adx-hero` / `adx-hero-copy` / `adx-date` / `adx-orbit`… | The hero band + orbit ring |
| `adx-grid` / `adx-column` / `adx-main` / `adx-side` | The two-column dashboard layout |
| `adx-panel` / `adx-section-head` / `adx-label` | Panels and their headers |
| `adx-task` / `adx-task-check` / `adx-task-body` / `adx-task-title` / `adx-task-meta` / `adx-pill` | Task rows (from `H.taskRow`) |
| `adx-project-row` / `adx-progress` / `adx-select` / `adx-work-index` | Project cards |
| `adx-nav` / `adx-nav-item` / `adx-project-create` | In-dashboard tab bars (note: distinct from the persistent `sbx-nav` rail) |
| `adx-empty` | Empty-state message |
| `adx-note-row` / `adx-note-title` / `adx-note-path` | Note list rows (Vault Health, Playground) |
| `adx-kind-switch` / `adx-kind-option` | Note/Task segmented control |
| `adx-academic-stats` / `adx-search-source` | Signal/stats panels |
| `sbx-nav` / `sbx-nav-item` / `sbx-nav-create` / `sbx-nav-tab` | The persistent navigation rail |
| `sbx-home-fab` | Floating "back to Home" button |
| `is-active` / `is-open` / `is-dragging` / `is-done` / `is-hot` | State modifiers |

All of these are styled in `.obsidian/snippets/atelier.css` unless noted. `lofi-scene.css` handles the workspace-specific classes.

---

## How dashboards load helpers

Every dashboard starts with:

```js
const H = new Function("dv", "require", "app", await app.vault.read((app.vault.getAbstractFileByPath("_core/helpers.js") || app.vault.getAbstractFileByPath(H.path("_core/helpers.js")))))(dv, require, app);
const { icon, open, sectionHead, empty, tabBar, ..., createNote, Notice } = H;
```

`H` is a namespace object built by a `return (function build({ dv, require, app }) {...})({ dv, require, app })` IIFE in `_core/helpers.js`. It is **not** a module — edit it directly and reload.

### The `H.*` API surface

| Export | What it does |
|---|---|
| `ROOT`, `path(rel)`, `q(folder)`, `isLab(p)`, `pagesIn(*)`, `labPages()` | Path/query resolution (dual-mode root) |
| `STATUS`, `ACTIVE_PROJECT`, `KIND_FRONTMATTER`, `values(type)`, `label(type, v)` | Status vocabularies + labels |
| `store.get(k, fb)` / `store.set(k, v)` | Sandbox storage (`sbx-` localStorage keys) |
| `icon(el, name)` / `open(path)` / `sectionHead(…)` / `empty(el, text)` | DOM helpers |
| `completeTask(task)` / `taskRow(parent, task)` | Task mutation + rendering |
| `form({title, fields})` | Native modal form → Promise of values |
| `createNote({kind, folder, template, extraFields})` | Template-based creation (staging-rename) |
| `patchFrontmatter(note, updates)` | Atomic frontmatter write |
| `logFocusSession({…})` / `captureToInbox(text, kind)` | Sessions/Inbox writers |
| `mountHome(el)` / `mountNav(el)` | Home FAB + nav rail |
| `kindToggle` / `tabBar` / `typeIcon` / `taskCategories` | Small UI components |
| `relative(date)` / `safeName(s)` / `clampPct(n)` / `Notice` | Utilities |

---

## Adding or editing a dashboard

To create a new dashboard note:

1. Copy the frontmatter block from any existing dashboard:
   ```yaml
   type: dashboard
   tags: [sandbox]
   cssclasses: [dashboard-layout, atelier-dashboard-page]
   ```
2. Add a `dataviewjs` block. Recommended skeleton:
   ```js
   const H = new Function(...);        // loader line (copy verbatim)
   const { icon, open, sectionHead, empty, tabBar, ... } = H;
   const root = dv.container.createDiv({ cls: "adx adx-enter" });
   H.mountHome(root); H.mountNav(root);
   // hero → grid → main + side → panels → render
   ```
3. Optional: add the note to `NAV` in `_core/helpers.js:305-318` and to the README table.

> Note: dashboards live in `00 dashboards/` (the folder that used to be the vault root). `NAV` and `mountHome` reference it explicitly (`open(path("00 dashboards/Home.md"))`), so if you rename the folder, update those open paths too — the dataviewjs inside is unaffected.

---

## Changing the navigation rail

The rail entries are hardcoded in `_core/helpers.js:305-318`:

```js
const NAV = [ ["home","Home","Home.md"], ["inbox","Inbox","Inbox.md"], ... ];
```

- **Add a link:** insert a `[icon, label, filename]` triple
- **Icons** are Lucide names passed to `setIcon`
- The **+ create** entry is appended automatically and opens `atelier-tools:new-note`
- Phone behavior: the rail becomes a draggable bubble; position saved per-device in `store` under `nav-pos`

---

## The Lo-fi Workspace

JavaScript for the rain/dust scene is vendored at `_core/vendor/three.min.js` (do not update via npm — replace the file manually and reload). The pixel-city backdrop is `_core/vendor/lofi-city.svg`, loaded as a CSS background by the workspace dashboard.

---

## Other knobs

| What you want | Where |
|---|---|
| Accent color | Settings → Appearance → accent (`#c8895b` default) |
| Default view mode | `.obsidian/app.json` → `defaultViewMode` (currently `preview`) |
| Attachments folder | `.obsidian/app.json` → attachment config → `Attachments/` |
| Startup page | Homepage plugin → open `Home.md` |
| Your name / daily focus | Click them on [[Home]] |
| Rainbow / lofi scene look | `.obsidian/snippets/lofi-scene.css` |
| Reading goal, focus timer, playlist | In-vault UI state, sandbox `store` |

---

## Gotchas

- **No hot reload** for `helpers.js` / CSS — reload Obsidian after edits
- **`defaultViewMode` is `preview`** — new notes open in reading mode, not source
- **Task completion bypasses the Tasks plugin** — `helpers.js:78-89` rewrites the markdown line directly, so Tasks-plugin event hooks won't fire
- **Dataview JS must be enabled** — Settings → Dataview → "JavaScript Queries"; without it nothing renders
- **Kiosk mode** — Atelier Tools toggles `body.atelier-lab-active` based on the active note's `dashboard-layout` cssclass (`main.js:159-174`); if a dashboard looks "not like a dashboard," check its cssclasses
- **Don't rename helpers variables** the dashboards destructure — they're wired by name (`H.icon`, `H.store`, …)