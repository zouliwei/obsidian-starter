# Vault Agent Guidelines

This document provides custom instructions and preferences for AI agents working in this Obsidian vault.

## Public Template: Zero Personal Information

This vault is a **public Obsidian starter template** and must contain **zero personal information**.

- **Check before every commit**: Before committing any change, scan the staged files (`git diff --cached`) for personal information. If you find any, remove or generalize it before you commit.

## Obsidian Vault Best Practices

- **Latest Edits**: Read the current file before editing and follow the user's latest changes. Older drafts and historical records must not override updated plans.
- **Note Organization**: Store both projects and notes in the flat `data/` folder; use the `type` property to distinguish them.
- **Wikilinks**: Prefer standard Obsidian wikilinks `[[Note Name]]` for internal note connections.
- **Frontmatter/Properties**: Preserve YAML frontmatter at the top of notes. Keep existing metadata intact unless explicitly asked to modify it.

## Command-Line Guidelines

- **Prefer GitHub CLI (**`gh`**)**: When performing GitHub-specific tasks, prefer using the GitHub CLI (`gh`) over other web-based or manual methods.
