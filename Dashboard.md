---
cssclasses:
  - compact-dashboard
---

## [[Tasks]]
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const today = dv.date("today").toMillis();
const isBenefit = (page, task) => page.file.name === "Credit Cards" && task.due;
const isScheduledNow = (task) => task.scheduled && (task.scheduled.toMillis() === today || (task.scheduled.toMillis() < today && !task.completed));
const isStartedNow = (task) => task.start && (task.start.toMillis() === today || (task.start.toMillis() < today && !task.completed));
const isListed = (page, task) => !isBenefit(page, task) && (isScheduledNow(task) || isStartedNow(task));
const isDuePassed = (task) => !task.completed && task.due && task.due.toMillis() < today;
const isScheduledPassed = (task) => !task.completed && task.scheduled && task.scheduled.toMillis() < today && !isDuePassed(task);
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
  const listed = "((has scheduled date) AND ((scheduled today) OR ((scheduled before today) AND (not done)))) OR ((has start date) AND ((starts today) OR ((starts before today) AND (not done))))";
  const duePassed = "((due before today) AND (not done))";
  const scheduledPassed = "((scheduled before today) AND (not done))";
  const renderTasks = async (extraFilters, cls) => {
    const query = [
      "```tasks",
      listed,
      ...extraFilters,
      `path regex matches /^(?:${paths})$/`,
      ...(group.key === "finance" ? ["((path regex does not match /Credit Cards\\.md$/) OR (no due date))"] : []),
      "sort by due AND scheduled AND priority AND filename",
      "hide edit button",
      "hide start date",
      "hide scheduled date",
      "hide due date",
      "hide recurrence rule",
      "hide task count",
      "hide toolbar",
      "```"
    ].join("\n");
    await MarkdownRenderer.render(app, query, dv.container.createDiv(cls ? { cls } : {}), dv.current().file.path, dv.component);
  };
  const tasks = [];
  for (const page of group.rows) {
    for (const task of page.file.tasks) {
      if (isListed(page, task)) tasks.push(task);
    }
  }
  if (tasks.some(isDuePassed)) await renderTasks([duePassed], "task-overdue");
  if (tasks.some(isScheduledPassed)) await renderTasks([scheduledPassed, `NOT ${duePassed}`], "task-reminder");
  if (tasks.some(t => !isDuePassed(t) && !isScheduledPassed(t))) await renderTasks([`NOT ${duePassed}`, `NOT ${scheduledPassed}`], "task-open");
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
const found = { month: [], quarter: [], half: [], year: [], once: [] };
if (card) {
  const text = await dv.io.load(card.file.path);
  for (const line of text.split("\n")) {
    const task = line.match(/^\s*- \[ \] (.*)$/);
    if (!task) continue;
    const start = task[1].match(/🛫 (\d{4}-\d{2}-\d{2})/);
    const scheduled = task[1].match(/⏳ (\d{4}-\d{2}-\d{2})/);
    const due = task[1].match(/📅 (\d{4}-\d{2}-\d{2})/);
    if (!start || start[1] > today || !scheduled || !due) continue;
    const recurrence = task[1].match(/🔁\s+(.+?)(?=\s+[🛫⏳📅]|$)/);
    const rule = recurrence ? recurrence[1].trim() : "";
    const interval = intervals.find(item => item.rule === rule);
    const state = due[1] <= today ? "due" : scheduled[1] < today ? "scheduled" : "open";
    found[interval ? interval.key : "once"].push({ state });
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
async function renderTasks(extraFilters, cls) {
  const query = [
    "```tasks",
    "not done",
    "has start date",
    "has due date",
    "((starts today) OR (starts before today))",
    ...extraFilters,
    `path regex matches /^${pathPattern}$/`,
    "sort by due",
    "hide postpone button",
    "hide start date",
    "hide edit button",
    "hide scheduled date",
    "hide due date",
    "hide recurrence rule",
    "hide task count",
    "hide toolbar",
    "```"
  ].join("\n");
  await MarkdownRenderer.render(app, query, dv.container.createDiv({ cls }), dv.current().file.path, dv.component);
}
async function renderGroup(entries, filters) {
  if (entries.some(entry => entry.state === "due")) await renderTasks([...filters, "((due today) OR (due before today))"], "finance-due");
  if (entries.some(entry => entry.state === "scheduled")) await renderTasks([...filters, "(scheduled before today)", "NOT ((due today) OR (due before today))"], "finance-scheduled");
  if (entries.some(entry => entry.state === "open")) await renderTasks([...filters, "NOT ((due today) OR (due before today))", "NOT (scheduled before today)"], "finance-open");
}
let shown = 0;
for (const interval of intervals) {
  if (!found[interval.key].length) continue;
  dv.header(3, interval.title);
  await renderGroup(found[interval.key], [`recurrence regex matches /^${interval.rule}$/`]);
  shown += found[interval.key].length;
}
if (found.once.length) {
  dv.header(3, "One-time");
  await renderGroup(found.once, ["is not recurring"]);
  shown += found.once.length;
}
if (!shown) dv.paragraph("No credit card benefits that can be used today or earlier.");
```

## [[Projects]]
```dataviewjs
const areas = ["growth", "fitness", "finance", "cooking", "travel", "lifestyle"];
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
const { MarkdownRenderer } = require("obsidian");
for (const group of groups) {
  dv.header(3, group.key.charAt(0).toUpperCase() + group.key.slice(1));
  const list = dv.container.createEl("ul", { cls: "dataview" });
  for (const page of group.rows) {
    const item = list.createEl("li", { cls: "directory-entry" });
    await MarkdownRenderer.render(app, page.file.link.toString(), item, dv.current().file.path, dv.component);
    const tags = Array.from(page.file.tags || []);
    if (tags.length) {
      const line = item.querySelector("p") || item;
      line.appendText(" ");
      line.createEl("span", { cls: "directory-tags", text: tags.join(" ") });
    }
  }
}
if (!pages.length) dv.paragraph("No active projects.");
```

## [[Notes]]
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
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
const { MarkdownRenderer } = require("obsidian");
for (const group of groups) {
  dv.header(3, group.key.charAt(0).toUpperCase() + group.key.slice(1));
  const list = dv.container.createEl("ul", { cls: "dataview" });
  for (const page of group.rows) {
    const item = list.createEl("li", { cls: "directory-entry" });
    await MarkdownRenderer.render(app, page.file.link.toString(), item, dv.current().file.path, dv.component);
    const tags = Array.from(page.file.tags || []);
    if (tags.length) {
      const line = item.querySelector("p") || item;
      line.appendText(" ");
      line.createEl("span", { cls: "directory-tags", text: tags.join(" ") });
    }
  }
}
if (!pages.length) dv.paragraph("No active notes.");
```
