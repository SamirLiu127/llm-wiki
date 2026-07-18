---
description: Wiki dashboard — stats, recent activity, pending reviews
---

# Wiki Dashboard

The user ran `/wiki $ARGUMENTS`.

## Procedure

1. **Check if wiki exists**: Look for `wiki/.llm-wiki/index.md` (or `$LLM_WIKI_ROOT/.llm-wiki/index.md`)
   - If not found: offer to run `scripts/init-wiki.sh` to bootstrap

2. **Read `.llm-wiki/index.md`** for the full page catalog

3. **Read `.llm-wiki/review.json`** for pending review items

4. **Read `.llm-wiki/cache/hot-cache.md`** if it exists (for session context)

5. **Present the dashboard**:

```
# Wiki Dashboard / 維基面板

**Total pages:** {N}
**Last updated:** {timestamp}
**Index status:** {fresh|stale — run /wiki-lint}

## Recent Activity / 最近活動
| Date | Operation | Title |
|------|-----------|-------|
... (from log if exists, or index modified dates)

## Page Types / 頁面類型
| Type | Count |
|------|-------|
| concept | N |
| article | N |
| person | N |
| synthesis | N |
| worklog | N |

## Pending Review / 待審核 ({N})
... (from review.json)

## Active Topics / 活躍主題
... (from hot-cache if available)
```

### Full workflow

Use `Skill("llm-wiki")` if deeper wiki operations are needed. See `SKILL.md` → `/wiki — Dashboard` for the complete procedure.
