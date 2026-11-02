---
type: note
area: cooking
status: active
tags:
source:
---
# 🍽️ Recipe Gallery

Recipes grouped by tag.

```dataviewjs
const groups = {};
for (const p of dv.pages('#recipe')) {
  let tags = (p.file.tags || []).filter(t => t !== '#recipe');
  if (!tags.length) tags = ['#uncategorized'];
  for (const t of tags) (groups[t] ??= []).push(p);
}
for (const t of Object.keys(groups).sort()) {
  dv.header(3, t.replace('#', ''));
  dv.list(groups[t].map(p => p.file.link));
}
```
