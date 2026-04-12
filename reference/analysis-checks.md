# Analysis Checks — Detection Patterns

## MCP Server Duplicates

These are known patterns of overlapping MCP servers. Check for any
combination of these in the user's configuration.

### Reasoning/Thinking servers
Servers that provide structured step-by-step reasoning:
- `mcp-reasoner`
- `sequential-thinking` (@modelcontextprotocol/server-sequential-thinking)
- `thinking-tool`

These overlap with Claude's native reasoning capabilities, especially in
Claude Opus and Sonnet. They add marginal value in Chat/Cowork where the
model's reasoning is already strong. May have more value in Claude Code CLI
for complex multi-step coding tasks.

### Web search servers
Servers that provide web search capabilities:
- `search1api`
- `brave-search`
- `tavily`
- `exa`
- `serpapi`

In Cowork and Claude Desktop, the native `WebSearch` tool is built-in and
requires no configuration. External search MCP servers add value only if
they provide features WebSearch doesn't: page crawling, news filtering by
source, trending topics, sitemap extraction.

Check if the user's search MCP has active/valid API credentials. Expired
keys or exhausted free tiers make the server pure overhead.

### File system access
Servers providing file operations on the local machine:
- `Desktop Commander` (read/write/search files, process management, Excel/DOCX editing)
- `Windows MCP` (file system + registry, clipboard, notifications, screenshots)
- `filesystem` (@modelcontextprotocol/server-filesystem)
- `mac-mcp` (macOS-specific operations)

Desktop Commander and Windows MCP overlap significantly on file operations
and process management. They complement each other on OS-specific features
(registry, notifications for Windows MCP; advanced search, Excel editing
for Desktop Commander). In Cowork with Computer Use active, screenshot
and clipboard features of Windows MCP/mac-mcp are less necessary.

### PDF tools
PDF manipulation servers:
- `PDF Tools - Fill, Analyze, Extract, View` (full-featured: forms, CSV extraction, bulk fill)
- `PDF By Anthropic` (basic: read and display)
- Native `pdf` skill (Cowork built-in)

"PDF By Anthropic" is a strict subset of "PDF Tools". If both are present,
the basic one is redundant.

### Image generation/editing
- `gimp-mcp` (requires GIMP running)
- `image-generation` servers (Replicate, DALL-E, etc.)
- Hugging Face connector (includes `gr1_z_image_turbo_generate`)

These serve different purposes (editing vs. generation) but check if the
user actually uses them.

## Plugin Duplicates

### Same plugin from multiple sources
Check `enabledPlugins` in settings.json for entries with the same base name
but different marketplace suffixes:
- `plugin-name@marketplace-a` + `plugin-name@marketplace-b`

This happens when a plugin exists in multiple marketplaces and the user
installed both. Keep the one from the more active/maintained source
(typically `claude-plugins-official`).

### Plugin vs. local skill overlap
Cross-reference enabled plugins with local skills in `~/.claude/skills/`.
Common overlaps:
- `code-review` plugin vs. local `code-review` skill
- `skill-creator` plugin vs. local `skill-creator` skill
- Generic plugin skills vs. user's custom-built equivalents

The user's custom skill is usually more tailored to their workflow.
Prefer keeping custom skills and disabling the generic plugin equivalent,
unless the plugin is actively maintained and the custom skill is stale.

### Plugin vs. plugin skill overlap
Remote plugins (knowledge-work-plugins) each bundle multiple skills.
Check for overlaps between plugin skills:
- `marketing:seo-audit` vs. user's custom `web-content-optimization` skill
- `marketing:content-creation` vs. user's `contextual-writing-assistant`
- `productivity:memory-management` vs. user's `project-memory-manager`

### Setup/onboarding plugins
These are useful once and then add unnecessary weight:
- `claude-code-setup` — initial Claude Code configuration
- `setup-cowork` — Cowork onboarding
- `claude-md-management` — CLAUDE.md creation/structuring

Recommend disabling after initial use. They can be re-enabled when needed.

## System Prompt Analysis

### Overlap detection between User Preferences and CLAUDE.md
Common areas of duplication:
- File naming conventions
- Skill consultation instructions
- Research/verification rules
- Content creation workflow
- Language preferences
- Navigation rules (cookies, etc.)

Strategy: each instruction should exist in exactly one place.
- **User Preferences**: personal identity, working relationship definition,
  high-level behavioral expectations
- **CLAUDE.md**: all operational instructions (file handling, skills,
  research, workflow, code conventions)

### Stale references
Check for instructions mentioning:
- MCP servers no longer in the configuration
- Tools or services that have been deprecated
- Skills that have been removed or renamed
- Workflows that reference unavailable features

### Token estimation
Rough estimate: 1 token ≈ 4 characters in English, ≈ 3 characters in
other Latin-alphabet languages. Count total characters in User Preferences
+ CLAUDE.md to estimate token cost per session.

## Skill Directory Health

### Orphan files
Files in the skills directory that aren't proper skills:
- `.zip` archives (export artifacts)
- Backup directories (`*-backup/`, `*-bak/`)
- Standalone files without SKILL.md
- Symlinks to external directories (check if target exists)

### Skill structure validation
A proper skill should have:
- `SKILL.md` file (required)
- Optional `reference/` or `references/` directory
- Optional `resources/` directory
- Optional `scripts/` directory

Flag skills that are just a single file or lack SKILL.md.

### Superseded skills
Check for version pairs:
- `skill-name` + `skill-name-v2`
- `skill-name` + `skill-name-agentic` (agentic version uses subagents)
- `skill-name` + `skill-name-enhanced`

The newer/enhanced version typically supersedes the original, but verify
with the user — the original may still be needed in environments where
the enhanced version doesn't work (e.g., agentic skills require Cowork
subagents and won't work in Chat).

## Resource Impact Reference

### RAM estimates by server type

| Server type | Typical RAM | Notes |
|-------------|-------------|-------|
| Node.js (local) | 40-50 MB | Direct execution |
| Node.js (npx) | 50-60 MB | Adds package download time at startup |
| Python | 25-40 MB | Depends on imported libraries |
| Python (venv) | 30-45 MB | Isolated environment |
| Docker container | 200-500 MB | Highest cost, includes container overhead |
| Python + ML libs | 100-300 MB | If importing torch, tensorflow, etc. |

### Startup latency
- Local installs: near-instant
- npx installs: 2-10 seconds (package resolution + download if not cached)
- Docker: 5-30 seconds (container startup)
- Python with venv: 1-3 seconds
