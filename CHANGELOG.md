# Changelog

All notable changes to this skill are documented in this file. The format
is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [2.0.0] - 2026-07-10

### Added

- **Operational health checks for MCP servers** (`reference/health-checks.md`):
  read-only verification per install type (local, npx/uvx, Docker),
  env var and placeholder detection, backing service checks, five-state
  classification (OK / degraded / broken / misconfigured / unverifiable).
  The audit never launches a server to test it.
- **Context cost analysis** (Context Cost Reference in
  `reference/analysis-checks.md`): token cost measured from the actual
  tool schemas and skill descriptions visible in the auditing session,
  not from flat per-tool averages. Deferred-loading awareness; declared
  assumptions, never presented as measurements, for servers not loaded.
- **Intervention plan**: consolidated cross-layer table in the report
  with explanation, difficulty (low / medium / high, scale defined) and
  expected impact, ordered quick wins first.
- **Interactive HTML report** (optional,
  `reference/report-html-template.html`): self-contained tabbed page,
  one tab per layer with the intervention plan as the last tab, sortable
  tables, status pills, no external dependencies — opens offline.
- Tools and Health columns in the MCP inventory and context cost rows in
  the impact summary of the report template.

### Fixed

- Added `~/.claude.json` (user-scope `mcpServers`) to the audited files:
  audits previously missed every user-scope Claude Code MCP server, the
  ones added via `claude mcp add`.
- Corrected the Chat column of the platform compatibility matrix: system
  prompt is User Preferences + Project prompts (not CLAUDE.md), skills
  come from claude.ai capabilities (not local/plugin skills).
- Frontmatter description now covers hooks, permissions and memory files,
  so requests like "audit my hooks" trigger the skill.
- Clarified the MEMORY.md load limit: first 200 lines or 25 KB, whichever
  comes first.

### Changed

- Cross-platform detection from Claude Desktop now uses the existence of
  `~/.claude.json` as the primary signal that Claude Code is in use;
  Code-specific keys in `settings.json` may be absent when the user has
  no hooks or custom permissions.

## [1.0.0] - 2026-04-13

### Added

- Initial release: 6-phase audit workflow (data collection, analysis,
  report, interactive validation, apply changes with backups, final
  report).
- Detection patterns for MCP server duplicates, plugin duplicates, skill
  ecosystem issues, and system prompt overlap
  (`reference/analysis-checks.md`).
- Report templates for initial and final reports
  (`reference/report-template.md`).
- Cross-platform audit (Claude Desktop ↔ Claude Code detection), hooks,
  permissions and memory files analysis (Claude Code), claude.ai support
  via session context.
