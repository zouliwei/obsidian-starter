---
cssclasses:
  - compact-dashboard
---

## Tomorrow
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const tomorrow = dv.date("tomorrow").toMillis();
const isBenefit = (page, task) => page.file.name === "Credit Cards" && task.due;
const today = dv.date("today").toMillis();
const isTomorrow = (page, task) => !isBenefit(page, task) && ((task.scheduled && task.scheduled.toMillis() === tomorrow) || (task.start && !task.completed && task.start.toMillis() <= tomorrow));
const isDuePassed = (task) => !task.completed && task.due && task.due.toMillis() < tomorrow;
const isScheduledPassed = (task) => !task.completed && task.scheduled && task.scheduled.toMillis() < tomorrow && !isDuePassed(task);
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/")).where(p => p.file.tasks.some(t => isTomorrow(p, t)));
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
  const listed = "((has scheduled date) AND (scheduled tomorrow)) OR ((has start date) AND (not done) AND ((starts tomorrow) OR (starts before tomorrow)))";
  const duePassed = "((due before tomorrow) AND (not done))";
  const scheduledPassed = "((scheduled before tomorrow) AND (not done))";
  const renderTasks = async (extraFilters, cls) => {
    const query = [
      "```tasks",
      listed,
      ...extraFilters,
      "((path regex does not match /Credit Cards\\.md$/) OR (no due date))",
      `path regex matches /^(?:${paths})$/`,
      "sort by status.type",
      "sort by priority",
      "sort by filename reverse",
      "hide start date",
      "hide scheduled date",
      "hide due date",
      "hide edit button",
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
      if (isTomorrow(page, task)) tasks.push(task);
    }
  }
  if (tasks.some(isDuePassed)) await renderTasks([duePassed], "task-overdue");
  if (tasks.some(isScheduledPassed)) await renderTasks([scheduledPassed, `NOT ${duePassed}`], "task-reminder");
  if (tasks.some(t => !isDuePassed(t) && !isScheduledPassed(t))) await renderTasks([`NOT ${duePassed}`, `NOT ${scheduledPassed}`], "task-open");
}
if (!pages.length) dv.paragraph("No tasks scheduled for tomorrow or already started.");
```

## Backlog
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/")).where(p => p.type === "project").where(p => p.file.tasks.some(t => !t.completed && !t.scheduled && !t.start));
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
  const query = [
    "```tasks",
    `path regex matches /^(?:${paths})$/`,
    "not done",
    "no scheduled date",
    "no start date",
    "sort by filename",
    "hide edit button",
    "hide recurrence rule",
    "hide task count",
    "hide toolbar",
    "```"
  ].join("\n");
  await MarkdownRenderer.render(app, query, dv.container.createDiv(), dv.current().file.path, dv.component);
}
if (!pages.length) dv.paragraph("No unscheduled tasks.");
```

## Scheduled
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const today = dv.date("today").toMillis();
const isBenefit = (page, task) => page.file.name === "Credit Cards" && task.due;
const isOpenDated = (page, task) => !task.completed && !isBenefit(page, task) && (task.scheduled || task.start);
const isDuePassed = (task) => task.due && task.due.toMillis() < today;
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/")).where(p => p.type === "project").where(p => p.file.tasks.some(t => isOpenDated(p, t)));
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
  const listed = [
    "not done",
    "((has scheduled date) OR (has start date))",
    "((path regex does not match /Credit Cards\\.md$/) OR (no due date))",
    `path regex matches /^(?:${paths})$/`
  ];
  const renderTasks = async (extraFilters, cls) => {
    const query = [
      "```tasks",
      ...listed,
      ...extraFilters,
      "sort by due AND scheduled AND start",
      "hide edit button",
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
      if (isOpenDated(page, task)) tasks.push(task);
    }
  }
  if (tasks.some(isDuePassed)) await renderTasks(["due before today"], "task-overdue");
  if (tasks.some(t => !isDuePassed(t))) await renderTasks(["NOT (due before today)"], "task-open");
}
if (!pages.length) dv.paragraph("No unfinished started or scheduled tasks.");
```

## Done
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const isBenefit = (page, task) => page.file.name === "Credit Cards" && task.due;
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/")).where(p => p.type === "project").where(p => p.file.tasks.some(t => t.completed && !isBenefit(p, t) && (t.scheduled || t.start)));
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
  const query = [
    "```tasks",
    "done",
    "((has scheduled date) OR (has start date))",
    "((path regex does not match /Credit Cards\\.md$/) OR (no due date))",
    `path regex matches /^(?:${paths})$/`,
    "sort by scheduled reverse AND start reverse",
    "hide edit button",
    "hide recurrence rule",
    "hide task count",
    "hide toolbar",
    "```"
  ].join("\n");
  await MarkdownRenderer.render(app, query, dv.container.createDiv(), dv.current().file.path, dv.component);
}
if (!pages.length) dv.paragraph("No completed started or scheduled tasks.");
```
