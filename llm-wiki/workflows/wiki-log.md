# /wiki-log — Capture the Current Discussion

**Purpose**: Land the *current session conversation* into a single dated
`worklog/` page — the missing counterpart to `/wiki-ingest`, which only ingests
**files**. `/wiki-log` ingests **the live discussion**.

**Invoked by**: `/wiki-log [--project <name>]` → SKILL.md routes here

---

## Design Rationale

`★ Insight ─────────────────────────────────────`

- `/wiki-ingest` turns *documents* into knowledge pages; `/wiki-log` turns the
  *work discussion happening right now* into a dated log. Different input, different
  page type — hence the fifth type `worklog` instead of polluting `article/`.
- **Append-only, single writer.** `/wiki-log` writes exactly one `worklog/` page
  and nothing else. The existing four types (concept/article/person/synthesis) are
  strictly read-only here: their pages are consulted only to resolve [[wikilinks]],
  never appended to or rewritten. This keeps the log from silently mutating curated
  knowledge.
- **Infer the landing spot; don't interrogate.** Bucket what was discussed into
  decisions / action items / discussion points / entities on your own. Only ask the
  user when something is genuinely ambiguous.

`─────────────────────────────────────────────────`

This behavior pattern is informed by obsidian-second-brain (MIT) `/obsidian-save`;
the logic is re-implemented for this fork, no upstream code was copied.

---

## Pre-Flight

### Step 0: Resolve Wiki Root

Determine the wiki root (from `SKILL.md` resolution rules):

1. `$LLM_WIKI_ROOT` environment variable
2. `wiki/` in current project
3. Ask user

If `$WIKI_ROOT/.llm-wiki/index.md` does not exist, offer to run
`scripts/init-wiki.sh` first (it creates the `worklog/` folder).

### Step 1: Read Context

Read these to understand the current state of the wiki:

1. **`WIKI_SCHEMA.md`** (skill-level) — the `worklog` page type + conventions
2. **`.llm-wiki/index.md`** (project-level) — all existing pages, for [[wikilink]] resolution
3. **`.llm-wiki/config.md`** (project-level) — `organize_by_type`, language default

### Step 2: Parse Arguments

- `--project <name>` — sets the `project` frontmatter value. If omitted, infer a
  reasonable project name from the discussion, or leave it empty.

---

## Extraction

### Step 3: Read the Current Session Conversation

The source is **this conversation**, not a file. Re-read the session so far and
work only from what was actually said — do not invent content.

### Step 4: Detect Language

Analyze the discussion content:

- Count CJK characters (Unicode U+4E00–U+9FFF) vs Latin characters
- `>70% CJK` → `zh`, `>70% Latin` → `en`, `30–70% mix` → `bilingual`

Store as `SESSION_LANGUAGE`.

### Step 5: Bucketed Extraction

Sort what was discussed into four buckets:

| Bucket | What goes here |
|--------|----------------|
| **決策 / Decisions** | Choices that were settled ("we'll use X", "dropped Y") |
| **待辦 / Action items** | Things to do — each becomes a `- [ ]` checkbox |
| **討論重點 / Discussion** | Key points, findings, context worth remembering |
| **實體 / Entities** | People, tools, concepts mentioned by name |

Keep it faithful. If nothing fits a bucket, leave that section with an empty
placeholder rather than padding it.

### Step 6: Resolve Entities to [[wikilinks]]

For each entity in Step 5:

1. Look it up in `index.md` (match slug, title, or `aliases`).
2. **If a page exists** → link to it as `[[slug]]` in the worklog's Links section.
   **Do not modify the target page** — no backlinks, no edits. Read-only.
3. **If no page exists** → write it as `[[slug]] (pending)` and add it to the
   report. **Never fabricate content and never auto-create the target page.**

---

## Generation

### Step 7: Derive Filename

- Slug: derive a short kebab-case slug from the session topic (see `WIKI_SCHEMA.md`
  → Slug Derivation). If the slug already exists for today, append `-2`, `-3`.
- Path: `$WIKI_ROOT/worklog/YYYY-MM-DD-{slug}.md`
  (or `$WIKI_ROOT/YYYY-MM-DD-{slug}.md` if `organize_by_type: false`).

### Step 8: Write the Worklog Page

1. Read `templates/worklog.md` from the skill directory.
2. Fill frontmatter:
   - `title`: descriptive title (bilingual format if `SESSION_LANGUAGE` is bilingual)
   - `type: worklog`
   - `language`: from Step 4
   - `created`, `modified`, `date`: today's date (`date` is the primary sort/filter key)
   - `project`: from Step 2
   - `based_on: [session]` — provenance is the live conversation
   - `tags: [worklog, ...]` — add topic tags as useful
   - `summary`: one-sentence description of the session
3. Fill the body sections from the four buckets:
   - `## 討論重點 / Discussion`
   - `## 決策 / Decisions`
   - `## 待辦 / Action items` — each item as `- [ ]`
   - `## 相關連結 / Links` — resolved `[[slug]]` and pending `[[slug]] (pending)`

This is the **only** file written. Do not touch any existing page.

### Step 9: Regenerate Index

Follow `workflows/ingest.md` Step 15 index regeneration procedure (it already scans
type subfolders, so `worklog/` pages are picked up). Worklog pages appear in the
All Pages table with `Type = worklog`.

### Step 10: Report

Present a clean summary:

```
# Worklog Saved / 工作日誌已保存

**File:** worklog/YYYY-MM-DD-{slug}.md
**Project:** {project or —}
**Language:** {en|zh|bilingual}

## Captured / 已擷取
- Decisions: {N}
- Action items: {N} ({open} open / {done} done)
- Discussion points: {N}
- Entities: {N}

## Links / 連結
| Entity | Target | Status |
|--------|--------|--------|
| ...    | [[slug]] | resolved |
| ...    | [[slug]] | pending — no page yet |
```

---

## Invariants (must hold)

- **Existing four types get zero writes.** Only `worklog/` is written.
- **No fabrication.** Unresolved links are marked `(pending)`, never invented.
- **Manual only.** No background/scheduled/hook trigger creates worklogs.
- **Provenance preserved.** `based_on: [session]`; original human input is never overwritten.

## Edge Cases

- **Nothing substantive discussed** — say so and skip writing rather than create an
  empty log.
- **Very long session** — summarize faithfully; prefer the most recent, concrete
  decisions and action items.
- **Ambiguous project** — leave `project` empty rather than guessing wrongly; the
  user can refine later.
