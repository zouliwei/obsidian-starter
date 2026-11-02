# Obsidian Starter

An Obsidian starter vault built around a task-driven Dashboard. Open `Dashboard.md` to see what's due today, your active projects and notes, and which credit card benefits are ready to use — all grouped by area of life.

## What's inside

### `data/` - the real data; other views only read from here

All projects and notes live in one flat `data/` folder. The `type` property (`project` or `note`) tells them apart, and `status` (`active` or `archived`) decides whether they appear on the Dashboard. Use `tags` for general labels, and `area` for the life area (e.g. `growth`, `finance`, `fitness`, `cooking`, `travel`, `lifestyle`).

A typical page starts like this:

```yaml
---
type: project
area: fitness
status: active
tags:
  - routine
---
```

The folder includes sample pages for every `area`, so the Dashboard has something to show on first open.

### `Dashboard.md` - the main dashboard view
- **Tasks**: tasks scheduled or started today, plus anything overdue, grouped by area. A task whose due date has passed is highlighted in red.
- **Credit Cards**: benefits from the `Credit Cards` page that can be used now, grouped by how often they renew (monthly, quarterly, half year, yearly, one-time).
- **Projects**: active projects (`type: project`, `status: active`), grouped by area.
- **Notes**: active notes (`type: note`, `status: active`), grouped by area.

### `system/views/` - other specialized views
- **Tasks**: Tomorrow, Backlog (unscheduled project tasks), Scheduled (open project tasks with a date), and Done.
- **Projects**: archived projects.
- **Notes**: archived notes.

### `system/templates/` - template files
- **Project**: `type: project`
- **Note**: `type: note`, with a `source` field
- **Recipe**: a cooking note with Ingredients and Operations sections

## Quick start

1. Clone or download this repo, then open the folder in Obsidian with **Open folder as vault**.
2. Install and enable these community plugins. The plugin folders are gitignored, so Obsidian won't have them yet:
   - **Dataview**, then turn on **Enable JavaScript queries** in its settings
   - **Tasks**
   - **Git** (optional, for syncing the vault with a repo)
3. If the Dashboard looks unstyled, go to **Settings > Appearance > CSS snippets** and enable `compact-dashboard`.
4. Open `Dashboard.md`.

## How tasks work

Tasks use the Tasks plugin's emoji format:

```markdown
- [ ] Workout 🔁 every 4 days ⏳ 2026-11-03
- [ ] Book flights 🛫 2026-11-01 ⏳ 2026-11-06 📅 2026-11-10
```

- ⏳ **Scheduled date**: for routine tasks on a specific date, or as a reminder to keep working on an already-started task (it sits between the start date and the due date).
- 🛫 **Start date**: for goal tasks. The task appears from that day until it's done.
- 📅 **Due date**: the deadline. Once it has passed, the task is shown in red.
- 🔁 **Recurrence**: when you complete a recurring task, Tasks creates the next one with new dates.

## Credit card benefits

The `Credit Cards` project holds one task per benefit, each with all three dates and a recurrence:

```markdown
- [ ] Dining $50 ([[Some Card]]) 🔁 every 3 months 🛫 2026-10-01 ⏳ 2026-11-30 📅 2026-12-31
```

A benefit appears in the Credit Cards section once its start date arrives. It is highlighted when the scheduled date (a reminder) arrives, and turns red when the due date (expiry) arrives. The section groups benefits by their 🔁 rule: `every month`, `every 3 months`, `every 6 months`, `every year`, or no recurrence for one-time benefits. These tasks are kept out of the Tasks section so they don't show up twice.

## Make it yours

- Delete or replace the sample pages in `data/`. Keep a page named `Credit Cards` if you want the Credit Cards section.
- To change the areas, edit the `areas` array at the top of each code block in `Dashboard.md` and in the pages in `system/views/`. The order of the array is the order the groups appear in.
- Adjust the templates in `system/templates/` to match the properties you use.

## For AI agents

`AGENTS.md` has instructions for AI agents working in the vault: where to put notes, how to handle frontmatter and links, and a rule to check for personal information before every commit.
