---
cssclasses:
  - compact-dashboard
---

## Tomorrow
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/")).where(p => p.file.tasks.some(t => t.scheduled && t.scheduled.toMillis() === dv.date("tomorrow").toMillis()));
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
    "scheduled tomorrow",
    "sort by priority AND filename",
    "hide scheduled date",
    "hide due date",
    "hide edit button",
    "hide recurrence rule",
    "hide task count",
    "hide toolbar",
    "```"
  ].join("\n");
  await MarkdownRenderer.render(app, query, dv.container.createDiv(), dv.current().file.path, dv.component);
}
if (!pages.length) dv.paragraph("No tasks scheduled for tomorrow.");
```

## Backlog
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/")).where(p => p.type === "project").where(p => p.file.tasks.some(t => !t.completed && !t.scheduled));
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
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/")).where(p => p.type === "project").where(p => p.file.tasks.some(t => !t.completed && t.scheduled));
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
    "has scheduled date",
    "sort by scheduled",
    "hide edit button",
    "hide recurrence rule",
    "hide task count",
    "hide toolbar",
    "```"
  ].join("\n");
  await MarkdownRenderer.render(app, query, dv.container.createDiv(), dv.current().file.path, dv.component);
}
if (!pages.length) dv.paragraph("No unfinished scheduled tasks.");
```

## Done
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/")).where(p => p.type === "project").where(p => p.file.tasks.some(t => t.completed && t.scheduled));
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
    "done",
    "has scheduled date",
    "sort by scheduled reverse",
    "hide edit button",
    "hide recurrence rule",
    "hide task count",
    "hide toolbar",
    "```"
  ].join("\n");
  await MarkdownRenderer.render(app, query, dv.container.createDiv(), dv.current().file.path, dv.component);
}
if (!pages.length) dv.paragraph("No completed scheduled tasks.");
```
