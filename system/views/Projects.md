---
cssclasses:
  - compact-dashboard
---

## Archived
```dataviewjs
const areas = ["growth", "finance", "fitness", "cooking", "travel", "lifestyle"];
const pages = dv.pages().where(p => p.file.folder !== "system" && !p.file.folder.startsWith("system/"))
  .where(p => p.type === "project" && p.status === "archived")
  .sort(p => {
    const number = p.file.name.match(/^\d+/);
    return [
      Array.from(p.file.tags || []).map(tag => tag.toLowerCase()).sort().join(","),
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
if (!pages.length) dv.paragraph("No archived projects.");
```
