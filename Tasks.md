---
type: dashboard
tags: [sandbox]
cssclasses: [dashboard-layout, atelier-dashboard-page]
---

```dataviewjs
(async () => {
const H = new Function("dv", "require", "app", await app.vault.read((app.vault.getAbstractFileByPath("_core/helpers.js") || app.vault.getAbstractFileByPath(H.path("_core/helpers.js")))))(dv, require, app);
const { icon, open, sectionHead, empty, tabBar, taskRow, store, typeIcon, taskCategories } = H;
const root = dv.container.createDiv({ cls: "adx adx-enter" });
H.mountHome(root);
H.mountNav(root);

const typeByPath = {};
H.labPages().forEach(p => { if (p.file && p.file.path) typeByPath[p.file.path] = p.type; });

const allTasks = H.labPages().file.tasks.array()
  .filter(t => H.isLab(t.path) && !t.path.includes("_templates/"))
  .sort((a, b) => (a.due?.ts || 9e15) - (b.due?.ts || 9e15));
const openTasks = allTasks.filter(t => !t.completed);
const doneWeek = allTasks.filter(t => t.completed && t.completion && t.completion >= dv.date("today") - dv.duration("7 days"));

const hero = root.createDiv({ cls: "adx-hero" });
const copy = hero.createDiv({ cls: "adx-hero-copy" });
copy.createDiv({ cls: "adx-date", text: "ATELIER LAB / TASK ATELIER" });
copy.createEl("h1", { text: "Everything in flight." });
copy.createDiv({ cls: "adx-focus", text: "Every open task in the Lab, grouped and filtered by the note it lives in. Tick one off and the source line flips to done." });
const orbit = hero.createDiv({ cls: "adx-orbit" }); orbit.createDiv({ cls: "adx-orbit-ring" });
orbit.createDiv({ cls: "adx-orbit-value", text: String(openTasks.length) });
orbit.createDiv({ cls: "adx-orbit-label", text: "open" });

let currentFilter = store.get("tasks-filter", "all");
const navBar = root.createDiv({ cls: "adx-nav adx-task-nav" });
const panel = root.createDiv({ cls: "adx-panel" });
const head = panel.createDiv({ cls: "adx-section-head" });
head.createDiv({ cls: "adx-label", text: "All tasks" });
const count = head.createEl("span", { cls: "adx-hint" });
const list = panel.createDiv();

const categories = taskCategories(openTasks, typeByPath);
const catDefs = [["list", "All", "all"], ...categories.map(([cat]) => [typeIcon(cat), cat.replace(/_/g, " "), cat])];
const prettyCat = c => c.replace(/_/g, " ");

const filtered = () => currentFilter === "all"
  ? openTasks
  : openTasks.filter(t => (typeByPath[t.path] || "other") === currentFilter);

const render = () => {
  list.innerHTML = "";
  const items = filtered();
  count.textContent = currentFilter === "all"
    ? `${items.length} open task${items.length === 1 ? "" : "s"} across ${categories.length} categor${categories.length === 1 ? "y" : "ies"}`
    : `${items.length} task${items.length === 1 ? "" : "s"} in ${prettyCat(currentFilter)}`;
  tabBar(navBar, catDefs, currentFilter, v => { currentFilter = v; store.set("tasks-filter", v); render(); });
  if (!items.length) return empty(list, currentFilter === "all" ? "No open tasks in the Lab." : `No open tasks in ${prettyCat(currentFilter)}.`);
  items.forEach(t => taskRow(list, t));
};

const donePanel = root.createDiv({ cls: "adx-panel" });
sectionHead(donePanel, "Completed this week", `${doneWeek.length} done`);
if (!doneWeek.length) { empty(donePanel, "No completions in the last seven days."); }
doneWeek.forEach(t => taskRow(donePanel, t));

render();
})();
```