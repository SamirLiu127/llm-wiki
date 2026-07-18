# /wiki-report — Recap Worklogs into a Synthesis Page

**Purpose**: Roll up `worklog/` pages over a date/project range into a single
`synthesis` page — a recap of what was decided, what is still open, and what was
discussed. This is the "匯報 / report" half of the worklog feature.

**Invoked by**: `/wiki-report [--since YYYY-MM-DD] [--project <name>] [--week]` →
SKILL.md routes here

---

## Design Rationale

`★ Insight ─────────────────────────────────────`

- A recap is a *saved query answer over worklogs*, which is exactly what the
  `synthesis` type already models (`query`, `based_on`, `confidence`). Reusing
  `synthesis` avoids schema bloat; the `based_on` chain points at the worklog pages
  it summarizes, keeping full provenance.
- Action-item status is derived deterministically from the checkbox syntax:
  `- [ ]` = open, `- [x]` = done. No separate task/kanban system is involved.

`─────────────────────────────────────────────────`

---

## Procedure

### Step 0: Resolve Wiki Root

Per `SKILL.md` rules (`$LLM_WIKI_ROOT` → `wiki/` → ask). If there are no
`worklog/` pages, tell the user there is nothing to report yet.

### Step 1: Parse Range / Filter Flags

| Flag | Effect |
|------|--------|
| `--since YYYY-MM-DD` | Keep worklogs with `date >= this` |
| `--project <name>` | Keep worklogs whose `project` equals `<name>` |
| `--week` | Shorthand: `--since` = today − 7 days |

Flags combine (AND). With no flags, include **all** worklogs.

### Step 2: Select Worklog Pages

1. List candidate pages: `$WIKI_ROOT/worklog/*.md`
   (or `$WIKI_ROOT/YYYY-MM-DD-*.md` with `type: worklog` if `organize_by_type: false`).
2. Read each page's frontmatter and keep those matching the flags on `date` / `project`.
3. Sort chronologically by `date`.

If nothing matches, report the empty range and stop (do not write a page).

### Step 3: Aggregate

Across the selected worklogs, gather:

- **Decisions** — merged list from every `## 決策 / Decisions` section, deduplicated.
- **Action items** — every `- [ ]` / `- [x]`, split into **open** vs **done**,
  with counts. Keep the source worklog for each item.
- **Discussion threads** — the recurring / significant `## 討論重點 / Discussion`
  points, grouped by theme.
- **Entities** — union of the [[wikilinks]] referenced across the worklogs.

### Step 4: Detect Language

Same rule as elsewhere: CJK vs Latin ratio → `en` / `zh` / `bilingual`. Prefer the
dominant language of the selected worklogs.

### Step 5: Write the Synthesis Page

Read `templates/synthesis.md`. Create
`$WIKI_ROOT/synthesis/synth-{YYYY-MM-DD}-{slug}.md`
(or flat root if `organize_by_type: false`), with `slug` describing the range,
e.g. `synth-2026-07-18-weekly-recap.md`.

Frontmatter:

```yaml
---
title: "Worklog Recap {range} / 工作彙報 {range}"
type: synthesis
language: {en|zh|bilingual}
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [worklog, report]
summary: "{One-sentence recap summary}"
query: "Recap of worklogs {range filter}"
based_on: [{worklog slugs...}]
confidence: high
---
```

Body — reuse the synthesis structure, repurposed for a recap:

```markdown
# Worklog Recap / 工作彙報

## Question / 問題
> Recap {range / project filter}

## Answer / 回答
[Narrative recap: what moved, what was decided, what's outstanding.]

## Decisions / 決策
- Decision — from [[worklog-slug]]

## Action Items / 待辦
**Open ({N}):**
- [ ] item — [[worklog-slug]]

**Done ({N}):**
- [x] item — [[worklog-slug]]

## Discussion Threads / 討論脈絡
- Theme — [[worklog-slug]], [[worklog-slug]]

## Entities / 涉及實體
- [[slug]] — how it came up

## Evidence / 證據
| Source Page | Key Point | Relevance |
|-------------|-----------|-----------|
| [[worklog-slug]] | ... | high |

## Confidence / 置信度: high
[Based directly on recorded worklogs.]
```

Every `based_on` entry must appear as a [[wikilink]] in the body so the provenance
chain is navigable.

### Step 6: Regenerate Index

Follow `workflows/ingest.md` Step 15 index regeneration procedure.

### Step 7: Optional — Plain-Markdown Deliverable

If the user wants something to share externally, also emit a standalone Markdown
recap (no frontmatter, no [[wikilinks]] — resolve them to plain text) to the path
the user names, or print it inline. This is optional and off by default.

### Step 8: Confirm

```
# Report Saved / 報告已保存

**File:** synthesis/synth-{date}-{slug}.md
**Range:** {since / project / week}
**Worklogs summarized:** {N}
**Action items:** {open} open / {done} done
**based_on:** {N} worklog pages
```

---

## Edge Cases

- **No worklogs in range** — report the empty range, write nothing.
- **`--week` with a stale clock** — `--week` is relative to today; state the computed
  since-date in the output so the range is unambiguous.
- **Report vs synthesis naming** — the recap is a real `synthesis` page; the
  `[worklog, report]` tags let `/wiki-query` and the index distinguish recaps from
  ordinary saved answers.
