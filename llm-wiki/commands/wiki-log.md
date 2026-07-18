---
description: Capture the current session discussion into a dated worklog page (append-only, never touches existing pages)
argument-hint: "[--project <name>]"
---

# Log the Current Discussion into the Wiki

The user ran `/wiki-log $ARGUMENTS`.

This captures **the current session conversation** — decisions, action items, key
discussion points, and mentioned entities — into a single dated `worklog/` page.
It is a **pure append-only writer**: it only writes to `worklog/`. Existing
concept/article/person/synthesis pages are **read-only** — the command resolves
[[wikilink]] targets against them but never appends to or rewrites them.

## Key Paths

- Wiki root: `./wiki/` (or `$LLM_WIKI_ROOT`)
- Worklog pages: `./wiki/worklog/`
- Skill directory: `~/.claude/skills/llm-wiki/`

## Procedure

### 1. Verify wiki exists

Check `./wiki/.llm-wiki/index.md`. If not found, offer to run `init-wiki.sh`.

### 2. Load the full workflow

Use `Skill("llm-wiki")` to load the complete procedure from `workflows/wiki-log.md`,
which covers:

- Reading the current session conversation
- Bucketed extraction: decisions / action items / discussion points / entities
- Bilingual detection (CJK ratio) → `language` field
- Creating `worklog/YYYY-MM-DD-{slug}.md` from `templates/worklog.md`
- Resolving [[wikilinks]] to existing pages (never modifying them); marking
  unresolved targets as pending — never fabricating or auto-creating pages
- Regenerating `index.md`
- Reporting what was captured, where it was written, and which links resolved

## Guarantees

- **Manual only** — no background or scheduled agent triggers this.
- **Existing four types untouched** — concept/article/person/synthesis pages get
  zero writes.
- **No fabrication** — unresolved [[wikilinks]] are flagged for review, not invented.
