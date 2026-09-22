---
type: dashboard
tags: [sandbox]
cssclasses: [dashboard-layout, atelier-dashboard-page]
---

```dataviewjs
(async () => {
const f = (app.vault.getAbstractFileByPath("_core/helpers.js") || app.vault.getAbstractFileByPath(H.path("_core/helpers.js")));
const H = new Function("dv", "require", "app", await app.vault.read(f))(dv, require, app);
const { icon, open, sectionHead, empty, relative, taskRow, store, captureToInbox, Notice, kindToggle, tabBar, typeIcon, taskCategories } = H;

const root = dv.container.createDiv({ cls: "adx adx-enter" });
H.mountNav(root);
const state = { name: store.get("name", "friend"), focus: store.get("focus", "Choose the work that changes you.") };
const now = new Date();
const greeting = now.getHours() < 5 ? "Still awake" : now.getHours() < 12 ? "Good morning" : now.getHours() < 18 ? "Good afternoon" : "Good evening";

// Data — sandbox scope only
const labTasks = H.labPages().file.tasks.array().filter(t => H.isLab(t.path) && !t.path.includes("_templates/"));
const openTasks = labTasks.filter(t => !t.completed);
const typeByPath = {};
H.labPages().forEach(p => { if (p.file && p.file.path) typeByPath[p.file.path] = p.type; });
const projects = dv.pages(H.q("Projects")).where(p => p.type === "project" && H.ACTIVE_PROJECT.includes(p.status)).array();
const books = dv.pages(H.q("Books")).where(p => p.type === "book" && p.status === "reading").array();
const applications = dv.pages(H.q("Applications"))
  .where(p => p.type === "job_application" && !["rejected", "withdrawn", "archived"].includes(p.status)).array();

// Hero
const hero = root.createDiv({ cls: "adx-hero" });
const copy = hero.createDiv({ cls: "adx-hero-copy" });
copy.createDiv({ cls: "adx-date", text: now.toLocaleDateString(undefined, { weekday: "long", month: "long", day: "numeric" }) });
const title = copy.createEl("h1");
title.createEl("span", { text: `${greeting}, ` });
const nameEl = title.createEl("span", { cls: "adx-editable", text: state.name, attr: { contenteditable: "true", spellcheck: "false" } });
title.createEl("span", { text: "." });
nameEl.onkeydown = e => { if (e.key === "Enter") { e.preventDefault(); nameEl.blur(); } };
nameEl.onblur = () => { state.name = nameEl.textContent.trim() || state.name; nameEl.textContent = state.name; store.set("name", state.name); };
const focus = copy.createDiv({ cls: "adx-focus", text: state.focus, attr: { contenteditable: "true", spellcheck: "true" } });
focus.onkeydown = e => { if (e.key === "Enter") { e.preventDefault(); focus.blur(); } };
focus.onblur = () => { store.set("focus", focus.textContent.trim()); };

const doneCount = labTasks.length - openTasks.length;
const score = labTasks.length ? Math.round(doneCount / labTasks.length * 100) : 0;
const orbit = hero.createDiv({ cls: "adx-orbit" });
orbit.createDiv({ cls: "adx-orbit-ring" });
orbit.createDiv({ cls: "adx-orbit-value", text: `${score}%` });
orbit.createDiv({ cls: "adx-orbit-label", text: "tasks completed" });

// Command bar
const command = root.createDiv({ cls: "adx-command" });
const search = command.createDiv({ cls: "adx-command-item adx-search" });
icon(search, "search"); search.createEl("span", { text: "Search the vault" }); search.createEl("kbd", { text: "Ctrl P" });
search.onclick = () => app.commands.executeCommandById("switcher:open");

const capture = command.createDiv({ cls: "adx-command-item adx-capture" });
icon(capture, "plus");
const captureInput = capture.createEl("input", { attr: { placeholder: "Capture to the inbox..." } });
let captureKind = store.get("capture-kind", "note");
const setCapturePlaceholder = () => captureInput.placeholder = `Capture ${captureKind === "task" ? "a task" : "a note"} to the inbox…`;
setCapturePlaceholder();
kindToggle(capture, captureKind, v => { captureKind = v; setCapturePlaceholder(); });
captureInput.onkeydown = async e => {
  if (e.key !== "Enter" || !captureInput.value.trim()) return;
  store.set("capture-kind", captureKind);
  await captureToInbox(captureInput.value, captureKind === "task" ? "task" : "thought");
  captureInput.value = "";
};

const createBtn = command.createDiv({ cls: "adx-command-item adx-create" });
icon(createBtn, "plus-circle"); createBtn.createEl("span", { text: "Create" });
createBtn.onclick = () => app.commands.executeCommandById("atelier-tools:new-note");

// Navigation
const dash = n => H.path("00 dashboards/" + n);
const nav = root.createDiv({ cls: "adx-nav" });
[["graduation-cap","Academics",dash("Academics.md")],["code-2","Programming",dash("Programming.md")],
 ["blocks","Projects",dash("Projects.md")],["check-square","Tasks",dash("Tasks.md")],
 ["book-open","Library",dash("Library.md")],
 ["briefcase-business","Career",dash("Jobs.md")],["microscope","Research",dash("Research.md")],
 ["heart-pulse","Life",dash("Life.md")],
 ["palette","Culture",dash("Culture.md")],["headphones","Lo-fi",dash("Lo-fi Workspace.md")],
 ["activity","Health",dash("Vault Health.md")]
].forEach(([i, l, p]) => { const n = nav.createDiv({ cls: "adx-nav-item" }); icon(n, i); n.createEl("span", { text: l }); n.onclick = () => open(p); });

const grid = root.createDiv({ cls: "adx-grid" });
const main = grid.createDiv({ cls: "adx-column adx-main" });
const side = grid.createDiv({ cls: "adx-column adx-side" });

// Pipeline
const pipeline = main.createDiv({ cls: "adx-panel adx-pipeline" });
sectionHead(pipeline, "Today / Pipeline", "All tasks", () => open(dash("Tasks.md")));
const pipeNav = pipeline.createDiv({ cls: "adx-nav adx-task-nav" });
const pipeCount = pipeNav.createDiv({ cls: "adx-hint" });
const pipeList = pipeline.createDiv({});
const pipeCats = taskCategories(openTasks, typeByPath);
let pipeFilter = store.get("home-pipe-filter", "all");
const renderPipeline = () => {
  const items = (pipeFilter === "all" ? openTasks : openTasks.filter(t => (typeByPath[t.path] || "other") === pipeFilter))
    .sort((a, b) => (a.due?.ts || 9e15) - (b.due?.ts || 9e15)).slice(0, 7);
  pipeCount.textContent = `${items.length} shown`;
  pipeNav.innerHTML = "";
  tabBar(pipeNav, [["list", "All", "all"], ...pipeCats.map(([c]) => [typeIcon(c), c.replace(/_/g, " "), c])], pipeFilter, v => { pipeFilter = v; store.set("home-pipe-filter", v); renderPipeline(); });
  pipeNav.appendChild(pipeCount);
  pipeList.innerHTML = "";
  if (!items.length) { empty(pipeList, "No open tasks in this category."); return; }
  items.forEach(t => taskRow(pipeList, t));
};
renderPipeline();

// Projects
const work = main.createDiv({ cls: "adx-section" });
sectionHead(work, "In motion", "All projects", () => open(dash("Projects.md")));
const workGrid = work.createDiv({ cls: "adx-work-grid" });
if (!projects.length) empty(workGrid, "No active projects yet.");
projects.slice(0, 4).forEach((p, i) => {
  const card = workGrid.createDiv({ cls: `adx-work-card tone-${i % 4}` });
  card.createDiv({ cls: "adx-work-index", text: String(i + 1).padStart(2, "0") });
  card.createDiv({ cls: "adx-work-title", text: p.file.name });
  card.createDiv({ cls: "adx-work-next", text: p.next_milestone || "Define the next milestone" });
  const prog = card.createDiv({ cls: "adx-progress" });
  prog.createDiv({ attr: { style: `width:${H.clampPct(p.progress)}%` } });
  card.onclick = () => open(p.file.path);
});

// Recently created notes (sandbox scope) — excludes dashboards and templates
const recent = main.createDiv({ cls: "adx-panel" });
sectionHead(recent, "Recently created");
H.labPages().filter(p => p.path && !p.path.includes("_templates/") && p.type !== "dashboard")
  .sort((p, q) => ((q.file.cday ? q.file.cday.toMillis() : 0) - (p.file.cday ? p.file.cday.toMillis() : 0))).slice(0, 6).forEach(p => {
    const row = recent.createDiv({ cls: "adx-note-row" }); icon(row, H.typeIcon(p.type));
    const body = row.createDiv({ cls: "adx-note-body" });
    body.createDiv({ cls: "adx-note-title", text: p.file.name });
    body.createDiv({ cls: "adx-note-path", text: p.file.folder || "Root" });
    row.createDiv({ cls: "adx-note-time", text: p.file.cday ? relative(p.file.cday) : "" });
    row.onclick = () => open(p.file.path);
  });

// Inbox preview — latest captures from daily inbox notes
const inboxPanel = side.createDiv({ cls: "adx-panel" });
sectionHead(inboxPanel, "Inbox", "Open inbox", () => open(dash("Inbox.md")));
(async () => {
  const days = dv.pages(H.q("Inbox"))
    .where(p => /^\d{4}-\d{2}-\d{2}$/.test(p.file.name))
    .sort(p => (p.file.day ? p.file.day.ts : 0), "desc")
    .array();
  const items = [];
  for (const p of days.slice(0, 3)) {
    const file = app.vault.getAbstractFileByPath(H.path(`Inbox/${p.file.name}.md`));
    if (!file) continue;
    const content = await app.vault.read(file);
    content.split("\n").forEach(line => {
      let m = line.match(/^\s*- \[( |x)\]\s+(.*)$/);
      let body = null, done = false;
      if (m) { body = m[2]; done = m[1] === "x"; }
      else { m = line.match(/^\s*- (?!\[)(.*)$/); if (m) body = m[1]; }
      if (!body) return;
      const clean = body.trim().replace(/\s*\*[^*]*\*$/, "").trim();
      if (clean) items.push({ text: clean, day: p.file.name, done });
    });
  }
  if (!items.length) { empty(inboxPanel, "Inbox is clear."); return; }
  items.slice(0, 5).forEach(it => {
    const row = inboxPanel.createDiv({ cls: "adx-note-row" });
    icon(row, it.done ? "check-circle" : "inbox");
    const body = row.createDiv({ cls: "adx-note-body" });
    body.createDiv({ cls: "adx-note-title", text: it.text });
    body.createDiv({ cls: "adx-note-path", text: it.day });
    row.onclick = () => open(dash("Inbox.md"));
  });
})();

// Shelf
const shelf = side.createDiv({ cls: "adx-panel" });
sectionHead(shelf, "Currently reading", "Open library", () => open(dash("Library.md")));
const resolveCover = (cover, app) => {
  try {
    if (!cover) return null;
    if (typeof cover === "object") cover = cover.path || cover.display;
    if (!cover) return null;
    let name = cover;
    const m = cover.match(/^\[\[(.+?)\]\]$/);
    if (m) name = m[1];
    const base = name.replace(/\.[a-z0-9]+$/i, "").toLowerCase();
    const file = (app.metadataCache.getFirstLinkpathDest(name, "") || app.metadataCache.getLinkpath?.(name))
      || app.vault.getFiles().find(f => f.path.toLowerCase().endsWith(name.toLowerCase()) || f.basename.toLowerCase() === base);
    return file ? app.vault.adapter.getResourcePath(file.path) : cover;
  } catch (_) { return cover; }
};
if (!books.length) empty(shelf, "The shelf is waiting.");
books.slice(0, 3).forEach(b => {
  const row = shelf.createDiv({ cls: "adx-book-row" });
  const coverSrc = resolveCover(b.cover, app);
  if (coverSrc) row.createEl("img", { attr: { src: coverSrc, alt: "" } });
  else { row.createDiv({ cls: "adx-cover-placeholder" }); icon(row.querySelector(".adx-cover-placeholder"), "book-open"); }
  const body = row.createDiv({ cls: "adx-book-body" });
  body.createDiv({ cls: "adx-book-title", text: b.title || b.file.name });
  body.createDiv({ cls: "adx-book-author", text: Array.isArray(b.authors) ? b.authors.join(", ") : b.author || "" });
  const meter = body.createDiv({ cls: "adx-progress" });
  meter.createDiv({ attr: { style: `width:${H.clampPct(b.progress)}%` } });
  row.onclick = () => open(b.file.path);
});

// Career funnel — normalized statuses
const career = side.createDiv({ cls: "adx-panel adx-career" });
sectionHead(career, "Career signal", "Open studio", () => open(dash("Jobs.md")));
const funnel = career.createDiv({ cls: "adx-funnel" });
H.STATUS.job.filter(([v]) => !["rejected", "withdrawn", "archived"].includes(v)).forEach(([value]) => {
  const count = applications.filter(a => a.status === value).length;
  const bar = funnel.createDiv({ cls: "adx-funnel-row" });
  bar.createEl("span", { text: H.label("job", value) });
  const track = bar.createDiv(); track.createDiv({ attr: { style: `width:${count ? Math.min(100, 24 + count * 18) : 3}%` } });
  bar.createEl("strong", { text: String(count) });
});
})(); 
```
