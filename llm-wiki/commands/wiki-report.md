---
description: Recap worklog pages over a date/project range into a synthesis report page
argument-hint: "[--since YYYY-MM-DD] [--project <name>] [--week]"
---

# Recap Worklogs into a Report

The user ran `/wiki-report $ARGUMENTS`.

This reads the `worklog/` pages matching a date/project range and rolls them up
into a single `synthesis` page: decisions made, action items (open vs done),
key discussion threads, and entities involved. It is **read-only over worklogs**
and only creates a new synthesis page.

## Key Paths

- Wiki root: `./wiki/` (or `$LLM_WIKI_ROOT`)
- Worklog pages: `./wiki/worklog/`
- Synthesis pages: `./wiki/synthesis/`
- Skill directory: `~/.claude/skills/llm-wiki/`

## Arguments

| Flag | Meaning |
|------|---------|
| `--since YYYY-MM-DD` | Include worklogs on or after this date |
| `--project <name>` | Only worklogs whose `project` matches |
| `--week` | Shorthand for the last 7 days (from today) |

With no flags, recap all worklogs.

## Procedure

### 1. Verify wiki exists

Check `./wiki/.llm-wiki/index.md`. If not found, there is nothing to report.

### 2. Load the full workflow

Use `Skill("llm-wiki")` to load the complete procedure from
`workflows/wiki-report.md`, which covers:

- Parsing the range/filter flags
- Selecting matching `worklog/` pages by `date` and `project`
- Aggregating decisions, action items (`- [ ]` open vs `- [x]` done), discussion
  threads, and entities
- Writing a `synthesis` page with `based_on: [worklog slugs...]`
- Regenerating `index.md`
- Optionally emitting a plain-Markdown deliverable for external sharing
