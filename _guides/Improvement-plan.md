---
type: guide
tags: [sandbox]
---

# Improvement-plan.md

> Confirmed gaps from a full vault audit, each with a recommendation, effort estimate, and the question it raises. Status reflects the **2026-09-22 build** — after structural changes, reload Obsidian (no hot reload) and re-run [[Vault Health]].

---

## 1. People.md is an empty stub

- **Current state:** `People.md` exists at the vault root with **zero content** (no frontmatter, no dashboard). It is **not** referenced in the navigation rail `NAV` (`_core/helpers.js:305-318`), so no dashboard knows it exists. The `People/` folder and both `_templates/Person.md` and `_templates/light/Person.md` are fully set up.
- **Recommendation:** Decide explicitly — either build a **People studio** (list people with relationship/context, surface interactions) or delete `People.md`. A half-built stub is the worst of both.
- **Effort:** Build ~30-45 min (copy the pattern from `Life.md`); delete ~1 min.
- **Question:** Do you want a People dashboard at all?

## 2. Dashboards at root vs README's "00 dashboards" claim

- **Current state:** README (`README.md:49-57`) documents a `00 dashboards/` folder. Actual dashboards live at the **vault root**. The name "remake" in this folder suggests this vault is the rebuilt/simplified successor — the README describes the *old target architecture*.
- **Recommendation:** Option A — update the README folder map to match reality (dashboards at root). Option B — actually move dashboards into a `_dashboards/` or `00 dashboards/` folder. Moving is safe (all links are resolved by filename via `mountNav`/`mountHome` which use `path(filename)`), but every dashboard's loader reads `_core/helpers.js` via a relative-path probe that already handles nested roots.
- **Effort:** Option A ~2 min; Option B ~20 min + reload + retest.
- **Question:** Keep dashboards at root (Option A) or consolidate into a folder (Option B)?

## 3. Root-level file clutter

- **Current state:** 15 markdown files at the vault root (14 dashboards + `People.md` stub) plus `README.md`, `START HERE.md`, `AGENTS.md`. The "domain folder" story is clean below root; the root itself is flat.
- **Recommendation:** If Option B above is chosen, this is solved in the same move. If dashboards stay at root, consider co-locating the guides (`_guides/` already does this) and leaving only the 15 dashboard files.
- **Effort:** Tied to #2.

## 4. Eight full templates lack light twins

- **Current state:** `Coding Problem`, `Code Snippet`, `Learning Path`, `Focus Session`, `Study Session`, `Daily`, `Weekly Review`, `Monthly Review` exist as full templates only. If an empty note of these types appears in a mapped folder, the skeleton engine falls through to `KIND_FRONTMATTER` synthesis — still typed, but minimal.
- **Recommendation:** Low priority today — these are mainly session/review/daily types that live outside mapped domain folders (Sessions + Inbox get their frontmatter from `logFocusSession()`/`captureToInbox()`, not light templates). Add light templates **only if** you create mapped folders for them in the future. See [[Template-guide]] for the 3-place checklist.
- **Effort:** ~5-10 min per type when needed.

## 5. `KIND_FRONTMATTER` is not exhaustive

- **Current state:** `_core/helpers.js:36-52` covers 14 kinds. `createNote()` falls back to a generic `{ type: "<kind>", status: "active" }` default (`helpers.js:182`) for kinds not in the map. All current create paths map to covered kinds, so this is latent — but a typo in `labTypesFor()` could silently produce an untyped/`status: active` note.
- **Recommendation:** When adding any new note kind, add its entry to `KIND_FRONTMATTER` in the same change. Optionally add a `console.warn` (already present in `createNote`, `helpers.js:180`) to the skeleton path too.
- **Effort:** negligible next time you touch either file.

## 6. Status vocabularies only cover project/job/book

- **Current state:** `STATUS` (`helpers.js:25-33`) defines canonical values for `project`, `job`, and `book`. `Vault Health` (`Vault Health.md:18-22`) only checks those three types — `course`, `assignment`, `research_question`, `life_area`, `work`, etc. use freeform `status` values with no validation.
- **Recommendation:** If you want Vault Health to police all types, extend `STATUS` for `course: active/paused/complete`, `assignment: planned/in_progress/submitted/graded`, `research_question: open/answered/abandoned`, `life_area: active/paused`, `work: want_to_experience/experiencing/finished/paused`. Confirm the values first, then add to `helpers.js` + `Vault Health`'s map.
- **Effort:** ~20 min + decide the vocabularies.
- **Question:** Worth it, or is freeform status for non-core types fine?

## 7. README plugin list drifts from actual plugins

- **Current state:** README lists **Tasks**, **Periodic Notes**, **Calendar**, **Style Settings** as required. Installed community plugins (`.obsidian/community-plugins.json`) are Atelier Tools, Dataview, Homepage, Templater — Tasks/Periodic/Calendar/Style aren't present, and Task completion is source-line mutation anyway (doesn't need the Tasks plugin). Templater is used for `folder_templates`, not for its templating UI.
- **Recommendation:** Trim README's plugin table to what's actually installed and needed: Dataview (JS queries), Templater (folder templates), Homepage, Atelier Tools. Mention Tasks/Periodic/Calendar/Style as optional legacy items if desired.
- **Effort:** ~10 min rewrite of `README.md:37-47`.

---

## Suggested execution order

1. **#1** (People stub) — one-line decision, then build or delete
2. **#2/#3** (root dashboards) — pick Option A or B
3. **#7** (README plugin table) — quick doc accuracy win
4. **#6** (status vocab) — only if you want stricter Vault Health
5. **#4/#5** — maintenance habits, not one-off tasks

Each item is independent. Do them in Obsidian's UI or ask the agent to execute — but always reload (no hot reload) and re-run [[Vault Health]] after structural changes.

---

## Status (2026-09-22 build)

| # | Item | Status |
|---|------|--------|
| 1 | People.md stub | **Done — deleted.** `People.md` removed; Home tile dropped. `People/` + templates remain for a future studio. |
| 2/3 | Dashboards at root / clutter | **Done — Option B.** All 14 dashboards moved to `00 dashboards/`. `NAV`, `mountHome`, Home tiles, Playground links and the Homepage plugin point at the folder. Root is now meta files only. |
| 4 | Missing light twins | Open — revisit only if mapped folders are created for session/review/daily types. |
| 5 | `KIND_FRONTMATTER` holes | Habit — extend whenever a new note kind is added. |
| 6 | Status vocabularies | **Decided: leave freeform** for non-core types. |
| 7 | README plugin list | **Done — table trimmed** to Dataview, Templater, Homepage, Atelier Tools; Tasks/Periodic/Calendar/Style noted as optional. |

Also shipped with this build (from Dashboard Review): "Lab"/"Atelier" visible strings removed vault-wide, Home hero now shows all-time tasks-completed %, Recently-touched replaced with a Recently-created panel, 12-week heatmap + focus-room panel removed, capture input spans the command grid with uniform radius, `.adx-nav` scrollbars hide until hover, Inbox row checkboxes are always visible, and the Academics tab bar sits below the hero.