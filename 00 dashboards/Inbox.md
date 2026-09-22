---
type: dashboard
tags: [sandbox]
cssclasses: [dashboard-layout, atelier-dashboard-page]
---

```dataviewjs
(async () => {
const f = (app.vault.getAbstractFileByPath("_core/helpers.js") || app.vault.getAbstractFileByPath(H.path("_core/helpers.js")));
const H = new Function("dv", "require", "app", await app.vault.read(f))(dv, require, app);
const { icon, open, sectionHead, empty, relative, store, captureToInbox, kindToggle, completeTask } = H;

const root = dv.container.createDiv({ cls: "adx adx-enter" });
H.mountNav(root);

const DAY = /^\d{4}-\d{2}-\d{2}$/;
const fmt = key => window.moment(key, "YYYY-MM-DD").format("ddd D MMM");
const todayKey = window.moment().format("YYYY-MM-DD");

const days = dv.pages(H.q("Inbox"))
  .where(p => DAY.test(p.file.name))
  .sort(p => (p.file.day ? p.file.day.ts : 0), "desc")
  .array()
  .map(p => p.file.name);

let selected = store.get("inbox-day", todayKey);
if (!days.includes(selected)) selected = (days.length ? days[0] : todayKey);
if (!days.includes(todayKey)) days.unshift(todayKey);

// ---- header ----
const hero = root.createDiv({ cls: "adx-panel" });
const heroHead = hero.createDiv({ cls: "adx-section-head" });
heroHead.createDiv({ cls: "adx-label", text: "INBOX" });
const heroStat = heroHead.createDiv({ cls: "adx-hint", text: `${days.length} day${days.length === 1 ? "" : "s"} of captures` });
const heroSub = hero.createDiv({ cls: "adx-hero-sub", text: "Capture from any dashboard lands in today's note. Select a day below to review it inline." });

// quick capture
const quick = hero.createDiv({ cls: "adx-capture" });
const quickInput = quick.createEl("input");
let captureKind = store.get("capture-kind", "note");
const setPlaceholder = () => quickInput.placeholder = `Capture a ${captureKind === "task" ? "task" : "note"} to today's inbox…`;
setPlaceholder();
kindToggle(quick, captureKind, v => { captureKind = v; setPlaceholder(); });
quickInput.onkeydown = async e => {
  if (e.key !== "Enter" || !quickInput.value.trim()) return;
  store.set("capture-kind", captureKind);
  await captureToInbox(quickInput.value, captureKind === "task" ? "task" : "thought");
  quickInput.value = "";
  render();
};

// ---- day selector (chips) ----
const chipRow = root.createDiv({ cls: "adx-nav adx-day-row" });
function renderChips() {
  chipRow.innerHTML = "";
  days.forEach((key, i) => {
    const chip = chipRow.createEl("button", {
      cls: `adx-nav-item${key === selected ? " is-active" : ""}`,
      attr: { title: key },
    });
    const isToday = key === todayKey;
    chip.createEl("span", { text: isToday ? "Today" : fmt(key) });
    chip.onclick = () => { selected = key; store.set("inbox-day", key); renderChips(); render(); };
  });
}

// ---- selected day detail ----
const panel = root.createDiv({ cls: "adx-panel" });
const detail = panel.createDiv({});
function render() {
  detail.innerHTML = "";
  const filePath = H.path(`Inbox/${selected}.md`);
  const file = app.vault.getAbstractFileByPath(filePath);
  const header = detail.createDiv({ cls: "adx-section-head" });
  const titleWrap = header.createDiv({});
  titleWrap.createDiv({ cls: "adx-label", text: fmt(selected) });
  titleWrap.createDiv({ cls: "adx-note-path", text: filePath });
  if (file) header.createEl("button", { cls: "adx-text-button", text: "Open note", attr: { title: filePath } }).onclick = () => open(H.path(`Inbox/${selected}.md`));

  if (!file) { empty(detail, "No captures for this day yet."); return; }

  app.vault.read(file).then(content => {
    const tasks = [];
    const notes = [];
    const filePath = H.path(`Inbox/${selected}.md`);
    const lines = String(content).split("\n");
    lines.forEach((line, idx) => {
      let m = line.match(/^\s*- \[( |x)\] (.*)$/);
      if (m) { tasks.push({ path: filePath, line: idx, text: m[2].trim(), done: m[1].toLowerCase() === "x" }); return; }
      m = line.match(/^\s*- (?!\[)(.*)$/);
      if (m) notes.push({ text: m[1].trim() });
    });
    const clean = t => t.replace(/\s*\*.*\*$/, "").trim();
    if (!tasks.length && !notes.length) { empty(detail, "Nothing captured this day yet."); return; }

    const group = label => detail.createDiv({ cls: "adx-inbox-group", text: label });
    if (tasks.length) {
      const head = group("Tasks");
      head.createEl("span", { cls: "adx-hint", text: `${tasks.length}` });
      const list = detail.createDiv({ cls: "adx-inbox-tasks" });
      tasks.forEach(r => {
        const row = list.createDiv({ cls: "adx-note-row" });
        const check = row.createEl("button", { cls: "adx-task-check", attr: { title: r.done ? "Completed" : "Complete task", "aria-label": "Complete task" } });
        icon(check, r.done ? "check-circle" : "circle");
        if (!r.done) check.onclick = async e => { e.stopPropagation(); if (await completeTask({ path: r.path, line: r.line })) render(); };
        const body = row.createDiv({ cls: "adx-note-body" });
        body.createDiv({ cls: "adx-note-title", text: clean(r.text) });
        if (r.done) body.createDiv({ cls: "adx-note-path", text: "done" });
        row.onclick = () => open(H.path(`Inbox/${selected}.md`));
      });
    }
    if (notes.length) {
      const head = group("Notes");
      head.createEl("span", { cls: "adx-hint", text: `${notes.length}` });
      const list = detail.createDiv({ cls: "adx-inbox-notes" });
      notes.forEach(r => {
        const row = list.createDiv({ cls: "adx-note-row" });
        icon(row, "sticky-note");
        const body = row.createDiv({ cls: "adx-note-body" });
        body.createDiv({ cls: "adx-note-title", text: clean(r.text) });
        row.onclick = () => open(H.path(`Inbox/${selected}.md`));
      });
    }
  });
}

renderChips();
render();
})();
```
