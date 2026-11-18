---
cssclasses:
  - compact-dashboard
---

## [[Tasks]]
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const today = dv.date("today").toMillis();
const isBenefit = (page, task) => page.file.name === "Credit Cards";
const isScheduledNow = (task) => task.scheduled && (task.scheduled.toMillis() === today || (task.scheduled.toMillis() < today && !task.completed));
const isStartedNow = (task) => task.start && (task.start.toMillis() === today || (task.start.toMillis() < today && !task.completed) || (task.completed && task.due && task.start.toMillis() <= today && task.due.toMillis() >= today));
const isListed = (page, task) => !isBenefit(page, task) && (isScheduledNow(task) || isStartedNow(task));
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/")).where(p => p.file.tasks.some(t => isListed(p, t)));
const groups = pages.groupBy(p => p.area || "Other")
  .sort(g => {
    const rank = areas.indexOf(g.key);
    return (rank < 0 ? "7-" : `${rank}-`) + g.key;
  });
dv.container.classList.add("dashboard-task-areas");
const trimBacklinks = () => {
  dv.container.querySelectorAll(".tasks-backlink a.internal-link").forEach(link => {
    const text = link.textContent || "";
    const cut = text.indexOf(" > ");
    if (cut !== -1) link.textContent = text.slice(0, cut);
  });
};
const backlinks = new MutationObserver(trimBacklinks);
backlinks.observe(dv.container, { childList: true, subtree: true, characterData: true });
dv.component.register(() => backlinks.disconnect());
const { MarkdownRenderer } = require("obsidian");
for (const group of groups) {
  dv.header(3, group.key.charAt(0).toUpperCase() + group.key.slice(1));
  const paths = group.rows.map(p => p.file.path.replace(/[.*+?^${}()|[\]\\/]/g, "\\$&")).join("|");
  const listed = "((has scheduled date) AND ((scheduled today) OR ((scheduled before today) AND (not done)))) OR ((has start date) AND ((starts today) OR ((starts before today) AND (not done)))) OR ((done) AND (has start date) AND (has due date) AND ((starts today) OR (starts before today)) AND ((due today) OR (due after today)))";
  // Keep due/scheduled in the DOM so CSS can mark overdue/reminder rows; they are visually hidden on .task-mixed.
  const query = [
    "```tasks",
    listed,
    `path regex matches /^(?:${paths})$/`,
    ...(group.key === "finance" ? ["path regex does not match /Credit Cards\\.md$/"] : []),
    "sort by status.type",
    "sort by scheduled",
    "sort by priority",
    "sort by filename reverse",
    "hide edit button",
    "hide start date",
    "hide recurrence rule",
    "hide task count",
    "hide toolbar",
    "```"
  ].join("\n");
  await MarkdownRenderer.render(app, query, dv.container.createDiv({ cls: "task-mixed" }), dv.current().file.path, dv.component);
}
if (!pages.length) dv.paragraph("No tasks started or scheduled for today or earlier.");
```

## [[Credit Cards]]
```dataviewjs
const today = dv.date("today").toFormat("yyyy-MM-dd");
const card = dv.page("Credit Cards");
const intervals = [
  { key: "month", title: "Monthly", rule: "every month" },
  { key: "quarter", title: "Quarterly", rule: "every 3 months" },
  { key: "half", title: "Half year", rule: "every 6 months" },
  { key: "year", title: "Yearly", rule: "every year" }
];
const found = { month: [], quarter: [], half: [], year: [], other: [] };
if (card) {
  const text = await dv.io.load(card.file.path);
  for (const line of text.split("\n")) {
    const task = line.match(/^\s*- \[ \] (.*)$/);
    if (!task) continue;
    const start = task[1].match(/🛫 (\d{4}-\d{2}-\d{2})/);
    const scheduled = task[1].match(/⏳ (\d{4}-\d{2}-\d{2})/);
    const due = task[1].match(/📅 (\d{4}-\d{2}-\d{2})/);
    if (!scheduled || scheduled[1] > today) continue;
    const recurrence = task[1].match(/🔁\s+(.+?)(?=\s+[🛫⏳📅]|$)/);
    const rule = recurrence ? recurrence[1].trim() : "";
    const interval = start && due ? intervals.find(item => item.rule === rule) : null;
    found[interval ? interval.key : "other"].push(1);
  }
}
const pathPattern = card ? card.file.path.replace(/[.*+?^${}()|[\]\\/]/g, "\\$&") : "";
const { MarkdownRenderer } = require("obsidian");
dv.container.classList.add("dashboard-task-areas");
const trimBacklinks = () => {
  dv.container.querySelectorAll(".tasks-backlink a.internal-link").forEach(link => {
    const text = link.textContent || "";
    const cut = text.indexOf(" > ");
    if (cut !== -1) link.textContent = text.slice(0, cut);
  });
};
const backlinks = new MutationObserver(trimBacklinks);
backlinks.observe(dv.container, { childList: true, subtree: true, characterData: true });
dv.component.register(() => backlinks.disconnect());
async function renderTasks(filters) {
  // Keep due/scheduled in the DOM so CSS can mark due/reminder rows; they are visually hidden on .task-mixed.
  const query = [
    "```tasks",
    "not done",
    "has scheduled date",
    "((scheduled today) OR (scheduled before today))",
    ...filters,
    `path regex matches /^${pathPattern}$/`,
    "sort by due",
    "hide postpone button",
    "hide start date",
    "hide edit button",
    "hide recurrence rule",
    "hide task count",
    "hide toolbar",
    "```"
  ].join("\n");
  await MarkdownRenderer.render(app, query, dv.container.createDiv({ cls: "task-mixed due-through-today" }), dv.current().file.path, dv.component);
}
let shown = 0;
for (const interval of intervals) {
  if (!found[interval.key].length) continue;
  dv.header(3, interval.title);
  await renderTasks(["has start date", "has due date", `recurrence regex matches /^${interval.rule}$/`]);
  shown += found[interval.key].length;
}
if (found.other.length) {
  dv.header(3, "Others");
  await renderTasks(["((no start date) OR (no due date) OR (is not recurring))"]);
  shown += found.other.length;
}
if (!shown) dv.paragraph("No credit card benefits with a reminder for today or earlier.");
```

## [[Projects]]
```dataviewjs
const areas = ["growth", "fitness", "finance", "cooking", "travel", "lifestyle"];
const emptyText = "No active projects.";
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/"))
  .where(p => p.type === "project" && p.status === "active")
  .sort(p => {
    const number = p.file.name.match(/^\d+/);
    const tags = Array.from(p.file.tags || []).map(tag => tag.toLowerCase());
    return [
      tags.length ? 0 : 1,
      tags.join(","),
      number ? 0 : 1,
      number ? -Number(number[0]) : 0,
      (/^[ -~]+$/.test(p.file.name) ? "0-" : "1-") + p.file.name.toLowerCase()
    ];
  });
const groups = pages.groupBy(p => p.area || "Other")
  .sort(g => {
    const rank = areas.indexOf(g.key);
    return (rank < 0 ? "7-" : `${rank}-`) + g.key;
  });
dv.container.classList.add("dashboard-directory");
const { setIcon } = require("obsidian");
const syncDataviewViews = () => {
  const index = app.plugins.plugins.dataview?.index;
  if (!index) return;
  const rev = index.revision;
  const walk = (comp) => {
    if (comp && typeof comp.lastReload === "number" && typeof comp.maybeRefresh === "function") comp.lastReload = rev;
    for (const child of comp?._children || []) walk(child);
  };
  app.workspace.iterateAllLeaves((leaf) => { if (leaf.view) walk(leaf.view); });
};
const archivePage = async (page, item) => {
  const file = app.vault.getAbstractFileByPath(page.file.path);
  if (!file) return;
  const list = item.parentElement;
  const heading = list?.previousElementSibling;
  item.remove();
  if (list && !list.children.length) {
    list.remove();
    if (heading && /^H[1-6]$/.test(heading.tagName)) heading.remove();
  }
  if (!dv.container.querySelector("li.directory-entry") && !dv.container.querySelector("p")) dv.paragraph(emptyText);
  const plugin = app.plugins.plugins.dataview;
  const before = plugin?.index?.revision;
  await app.fileManager.processFrontMatter(file, (fm) => { fm.status = "archived"; });
  const markCurrent = async () => {
    const start = Date.now();
    while (plugin?.index && plugin.index.revision === before && Date.now() - start < 2000) await new Promise((r) => setTimeout(r, 16));
    syncDataviewViews();
  };
  await markCurrent();
  const delay = (plugin?.settings?.refreshInterval ?? 2500) + 50;
  window.setTimeout(syncDataviewViews, Math.min(delay, 400));
  window.setTimeout(syncDataviewViews, delay);
};
for (const group of groups) {
  dv.header(3, group.key.charAt(0).toUpperCase() + group.key.slice(1));
  const list = dv.container.createEl("ul", { cls: "dataview" });
  for (const page of group.rows) {
    const item = list.createEl("li", { cls: "directory-entry" });
    const line = item.createEl("p");
    line.createEl("a", { cls: "internal-link", text: page.file.name, href: page.file.path, attr: { "data-href": page.file.path } });
    const tags = Array.from(page.file.tags || []);
    if (tags.length) {
      line.appendText(" ");
      line.createEl("span", { cls: "directory-tags", text: tags.join(" ") });
    }
    const archive = line.createEl("a", { cls: "directory-archive", attr: { href: "#", title: "Archive", "aria-label": "Archive" } });
    setIcon(archive, "archive");
    archive.addEventListener("click", async (event) => {
      event.preventDefault();
      event.stopPropagation();
      if (archive.dataset.busy) return;
      archive.dataset.busy = "1";
      try { await archivePage(page, item); }
      finally { delete archive.dataset.busy; }
    });
  }
}
if (!pages.length) dv.paragraph(emptyText);
```

## [[Notes]]
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const emptyText = "No active notes.";
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/"))
  .where(p => p.type === "note" && p.status === "active")
  .sort(p => {
    const number = p.file.name.match(/^\d+/);
    const tags = Array.from(p.file.tags || []).map(tag => tag.toLowerCase()).sort();
    return [
      tags.length ? 0 : 1,
      tags.join(","),
      number ? 0 : 1,
      number ? -Number(number[0]) : 0,
      (/^[ -~]+$/.test(p.file.name) ? "0-" : "1-") + p.file.name.toLowerCase()
    ];
  });
const groups = pages.groupBy(p => p.area || "Other")
  .sort(g => {
    const rank = areas.indexOf(g.key);
    return (rank < 0 ? "7-" : `${rank}-`) + g.key;
  });
dv.container.classList.add("dashboard-directory");
const { setIcon } = require("obsidian");
const syncDataviewViews = () => {
  const index = app.plugins.plugins.dataview?.index;
  if (!index) return;
  const rev = index.revision;
  const walk = (comp) => {
    if (comp && typeof comp.lastReload === "number" && typeof comp.maybeRefresh === "function") comp.lastReload = rev;
    for (const child of comp?._children || []) walk(child);
  };
  app.workspace.iterateAllLeaves((leaf) => { if (leaf.view) walk(leaf.view); });
};
const archivePage = async (page, item) => {
  const file = app.vault.getAbstractFileByPath(page.file.path);
  if (!file) return;
  const list = item.parentElement;
  const heading = list?.previousElementSibling;
  item.remove();
  if (list && !list.children.length) {
    list.remove();
    if (heading && /^H[1-6]$/.test(heading.tagName)) heading.remove();
  }
  if (!dv.container.querySelector("li.directory-entry") && !dv.container.querySelector("p")) dv.paragraph(emptyText);
  const plugin = app.plugins.plugins.dataview;
  const before = plugin?.index?.revision;
  await app.fileManager.processFrontMatter(file, (fm) => { fm.status = "archived"; });
  const markCurrent = async () => {
    const start = Date.now();
    while (plugin?.index && plugin.index.revision === before && Date.now() - start < 2000) await new Promise((r) => setTimeout(r, 16));
    syncDataviewViews();
  };
  await markCurrent();
  const delay = (plugin?.settings?.refreshInterval ?? 2500) + 50;
  window.setTimeout(syncDataviewViews, Math.min(delay, 400));
  window.setTimeout(syncDataviewViews, delay);
};
for (const group of groups) {
  dv.header(3, group.key.charAt(0).toUpperCase() + group.key.slice(1));
  const list = dv.container.createEl("ul", { cls: "dataview" });
  for (const page of group.rows) {
    const item = list.createEl("li", { cls: "directory-entry" });
    const line = item.createEl("p");
    line.createEl("a", { cls: "internal-link", text: page.file.name, href: page.file.path, attr: { "data-href": page.file.path } });
    const tags = Array.from(page.file.tags || []);
    if (tags.length) {
      line.appendText(" ");
      line.createEl("span", { cls: "directory-tags", text: tags.join(" ") });
    }
    const archive = line.createEl("a", { cls: "directory-archive", attr: { href: "#", title: "Archive", "aria-label": "Archive" } });
    setIcon(archive, "archive");
    archive.addEventListener("click", async (event) => {
      event.preventDefault();
      event.stopPropagation();
      if (archive.dataset.busy) return;
      archive.dataset.busy = "1";
      try { await archivePage(page, item); }
      finally { delete archive.dataset.busy; }
    });
  }
}
if (!pages.length) dv.paragraph(emptyText);
```
