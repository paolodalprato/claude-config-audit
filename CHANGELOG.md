# Changelog

All notable changes to this skill are documented in this file. The format
is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [2.2.2] - 2026-07-14

### Fixed

- **The 2.2.1 corrections now reach the operative files.** The v2.2.1
  fixes (plugins available in Chat, "Cowork Global Instructions"
  terminology) had been applied to README.md and
  `reference/hierarchy-checks.md` only: the environment matrix in
  SKILL.md and the Cowork section of the report template still carried
  the stale statements — and those are the files the model executes,
  so generated reports repeated them. Both aligned; the frontmatter
  version now tracks releases again.
- **Numeric sorting in the HTML report handles localized figures.**
  The column sort parsed "1.200" (Italian thousands) and "1,200"
  (English thousands) as 1.2, misordering token and RAM columns.
  Thousands separators are now stripped before comparison; a remaining
  comma is treated as a decimal comma.
- **User questions reach the HTML report.** The markdown report closes
  with "Questions for You", but the HTML template had no slot for it:
  the plan tab now includes a `{{QUESTIONS_CONTENT}}` placeholder.
- **Hierarchy findings have a declared home in the report.**
  `hierarchy-checks.md` prescribed one ordering that mixed within-stack
  findings (same-file contradictions, misplacement) with cross-stack
  ones (coverage gaps, drift), while the template only had slots for
  the latter. Now: within-stack findings open each environment's
  issues section; cross-stack findings live in the cross-environment
  section, coverage gaps first, then drift, then the override map.
- **RAM is no longer summable across environments.** Chat and Cowork
  load the same server processes from claude_desktop_config.json; the
  Resource Impact Summary now marks them as shared — the machine pays
  each process once.
- Unused-skill detection reworded: usage is not observable from
  configuration alone, so apparent non-use is a candidate to confirm
  in Phase 4 validation, never a standalone finding.
- Token estimation in SKILL.md now states the ÷ 3 divisor for
  non-English Latin-alphabet text (previously only in the Context Cost
  Reference), avoiding a ~33% overestimate on e.g. Italian instruction
  files.
- Renamed the leftover `{{PLATFORM*}}` placeholders in the HTML
  template to environment terminology (`{{AUDIT_ENVIRONMENT}}`,
  `{{ENVIRONMENTS_ANALYZED}}`); "In Cowork and Claude Desktop" → "In
  Chat and Cowork" in `analysis-checks.md`.

### Added

- **Non-file instruction layers** (Phase 1): User Preferences and the
  Cowork Global Instructions are not files on disk and are observable
  only from a session of their own environment. When auditing from
  elsewhere (typically Code), ask the user to paste them; if
  unavailable, the hierarchy analysis runs on the layers present and
  marks its cross-environment findings as partial.
- **Chat and Cowork memory acknowledged.** The environment matrix no
  longer claims memory is Code-only: Chat feeds a shared project
  memory that Cowork mounts read-only next to its own `.auto-memory/`
  directory. The file-level memory checks remain Code-only (project
  workspace memory is reachable only from a session in that project),
  and the report must state this scope limit.

## [2.2.1] - 2026-07-12

### Fixed

- **Plugins now available in Chat.** The environment matrix listed
  plugins as Cowork/Code only; Anthropic later enabled plugins on the
  Chat surface (their bundled skills run there, while hooks and
  sub-agents remain Cowork/Code). Chat column corrected.
- **Precedence wording tightened.** Clarified that user layers can
  override the system-managed defaults; what they cannot override are
  the non-negotiable boundaries, where the system part wins by
  construction. The previous phrasing implied the system wins on
  everything, including the defaults users routinely override.
- **Cowork instruction terminology corrected.** The Cowork global
  instruction layer is the Global Instructions field (Settings →
  Cowork), not a CLAUDE.md file. Updated the environment matrix and
  `reference/hierarchy-checks.md` (stack table, instruction-sources
  line, stack-equivalence example) accordingly. CLAUDE.md files remain
  the Code mechanism.

## [2.2.0] - 2026-07-11

### Changed

- **Scope declared: the audit covers Claude Desktop** and its three
  environments — Chat, Cowork, and Code.
- **The environment is the unit of analysis.** Components (files,
  servers, skills, plugins) are verified once, but duplicates, context
  cost, and coherence are judged per environment, where they are paid:
  a duplicate inside one environment is a finding, the same component
  in two environments usually is not.
- **Stack equivalence replaces file equality** in the hierarchy
  analysis: two global CLAUDE.md files serving different environments
  are not copies. Expected differences that compensate a missing level
  (e.g. User Preferences, which the Code environment never receives)
  are declared compensation, not drift; drift is divergence in the
  shared core. Recommended maintenance model: shared core + marked
  compensation block.
- Report restructured: component inventory, one section per
  environment, cross-environment checks, intervention plan. HTML
  template tabs follow the same structure.

### Fixed

- The offer of the interactive HTML report is now an explicit,
  non-optional step closing Phase 3. Test runs showed it was skipped
  when phrased as prose at the end of the phase.
- Photograph rule hardened: the audit must not look for previous
  reports, read them, or offer a comparison. Test runs showed it found
  old reports on disk and asked whether to compare.
- Footprint table: the user-managed total now explicitly includes the
  installed components (skill descriptions and tool definitions), which
  are as user-controlled as the instruction files — making that count
  visible is one of the audit's purposes. Terminology clarified: "tool
  definition (schema)" defined in place, plugins explicitly mentioned
  as a source of skills and connectors.

### Added

- **System prompt footprint**: the report opens with the total size of
  the auditing session's system prompt, split into the system-managed
  part (fixed: product and tool instructions, schemas, metadata) and the
  user-managed part (instructions and installed components — the
  audit's actionable surface). The documentation explains the
  distinction and links Anthropic's published system prompts.
- **Photograph rule**: the report documents the current state; no
  comparisons with previous audits unless explicitly requested.
- **Verification discipline**: open a folder's SKILL.md before
  classifying it as leftover; cite the exact file and JSON key read,
  never one reconstructed from memory.

## [2.1.0] - 2026-07-11

### Added

- **Instruction hierarchy analysis** (`reference/hierarchy-checks.md`):
  effective configuration stack per platform (what each session actually
  receives, including the fact that Claude Code never sees User
  Preferences), and a four-outcome classification of cross-level
  differences — misplacement, drift, coverage gap, presumed
  specialization. A rewrite test ("in general X, but in this context Y")
  separates true conflicts from legitimate per-project overrides, which
  go to an informational override map instead of the issues list.
  Framework tested on a real multi-platform configuration before being
  documented.
- Effective stack and override map subsections in the report template.
- Documented the MCP-source asymmetry between the Claude Code CLI and
  the Code tab in the Desktop app: the latter also loads
  `claude_desktop_config.json`. Noted in the platform matrix and in the
  health checks (a server visible in one frontend and absent in the
  other is not necessarily broken).

## [2.0.1] - 2026-07-10

### Added

- Explicit language rule: all user-facing output (markdown report, HTML
  report, validation questions, recommendations) is written in the
  language the user is using in the conversation. Templates define
  structure, not language; file paths, configuration keys and technical
  identifiers stay untranslated.

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
