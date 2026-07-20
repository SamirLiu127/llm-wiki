# Changelog

All notable changes to LLM Wiki will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `worklog` page type — a fifth, independent type for dated logs of work discussions (`worklog/YYYY-MM-DD-{slug}.md`), with `templates/worklog.md` and a full spec in `WIKI_SCHEMA.md`
- `/wiki-log` command — captures the current session conversation (decisions, action items, discussion points, entities) into a single dated worklog page. Append-only: writes only to `worklog/`, links to existing pages read-only via [[wikilinks]] (unresolved links marked `(pending)`, never fabricated), and regenerates the index. Manual-only, no background agent
- `/wiki-report` command — recaps worklog pages over a `--since` / `--project` / `--week` range into a `synthesis` page, counting open vs done action items and chaining `based_on` back to the source worklogs
- `commands/wiki.md` — dedicated `/wiki` dashboard slash command file (previously undocumented as "skill auto-registration" and resolved to `/llm-wiki` instead)
- `organize_by_type` config option — pages are stored under `article/`, `concept/`, `person/`, `synthesis/` subfolders by default, with the flat layout still available by setting it to `false`

### Changed

- All simplified Chinese text in templates, schema, and workflow docs converted to Traditional Chinese (Taiwan usage)
- Structural lint scripts (`find-broken-links.sh`, `find-orphans.sh`, `validate-frontmatter.sh`, `check-stale.sh`) now scan one level of subfolders so they work with both the flat and type-organized layouts

## [0.1.0] — 2026-05-03

### Added

- Initial release of LLM Wiki skill for Claude Code
- Two-phase source ingestion (`/wiki-ingest`) with SHA-256 idempotency
- Index-first knowledge retrieval (`/wiki-query`) for O(1) lookup
- Health check system (`/wiki-lint`) with quick (bash) and full (LLM) modes
- Answer-to-synthesis persistence (`/wiki-save`) for compounding knowledge
- D3.js knowledge graph visualization (`/wiki-graph`)
- Review queue processing (`/wiki-review`) for contradiction/quality management
- Wiki dashboard (`/wiki`)
- Bilingual support (en/zh/bilingual) with CJK auto-detection
- Session lifecycle hooks (start/stop) with hot-cache for context continuity
- Page templates for concept, article, person, and synthesis types
- Project setup script (`setup-project.sh`) with optional hooks configuration
- Wiki initialization script (`init-wiki.sh`)
- Global installation script (`install.sh`)
- Quickstart script (`quickstart.sh`) with demo content
- Uninstall script (`uninstall.sh`)
- Demo source files (Greek mythology)
- CI pipeline (ShellCheck + markdownlint + integration tests)
- Local CI runner (`scripts/ci-local.sh`)
- Community files (CoC, Contributing, Security, Support, PR/Issue templates)

### Fixed

- CI failures in initial workflow configuration
- Portability issues for non-Linux environments
