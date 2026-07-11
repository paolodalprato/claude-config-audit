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
model's reasoning is already strong. May have more value in the Code
environment for complex multi-step coding tasks.

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

**Two data sources for plugins (Claude Code):**
- `settings.json` → `enabledPlugins`: the enabled/disabled state (boolean per
  plugin). This is the authoritative source for what's active.
- `installed_plugins.json` → `plugins`: installation metadata (version,
  install date, marketplace, git commit SHA). Use this to check for stale
  versions, abandoned plugins, or plugins installed but no longer enabled.

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

For the cross-level hierarchy analysis — effective stack per platform,
misplacement, drift vs. legitimate specialization, coverage gaps — see
`hierarchy-checks.md`. The checks below cover content-level issues
within and between the two global layers.

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

## Hooks Analysis (Claude Code only)

Hooks are shell commands in `settings.json` triggered on events. They run
outside Claude's control and can affect performance and behavior.

### Duplicate or overlapping hooks
- Multiple hooks on the same event (`PreToolUse`, `PostToolUse`, etc.)
  running similar commands (e.g., two linters on the same file types)
- Global hooks duplicated at project level with identical commands

### Broken hooks
- Hooks referencing scripts or executables that don't exist on the system
- Hooks referencing paths that are environment-specific (e.g., hardcoded
  absolute paths that won't work on another machine)

### Performance risk
- Hooks with commands that may be slow (network calls, large file scans)
  and lack a timeout — these block tool execution
- Hooks that spawn background processes without cleanup

### Conflict detection
- Hooks that contradict each other (e.g., one hook formats code on save,
  another reverts formatting)
- Global hooks that conflict with project-level hooks on the same event

## Permissions Analysis (Claude Code only)

Permissions in `settings.json` control which tools Claude can use without
asking for confirmation.

### Overly broad rules
- `"allow": ["Bash"]` without path or command restrictions gives Claude
  unrestricted shell access. Flag for user awareness (not necessarily wrong,
  but the user should know).
- `"allow": ["Edit"]` or `"allow": ["Write"]` without path restrictions

### Stale permissions
- Allow/deny rules referencing MCP server tools that are no longer installed
  (e.g., `mcp__old-server__tool_name` when `old-server` has been removed)
- Rules that were added for temporary debugging and forgotten

### Conflicts
- A tool appearing in both allow and deny lists (deny takes precedence, but
  the allow rule is dead weight)
- Project-level permissions contradicting global permissions without clear
  intent

## Memory Files Analysis (Claude Code only)

Memory files live in `~/.claude/projects/*/memory/` with an index at
`MEMORY.md` in the same directory. They persist context across sessions.

### Stale memories
- Memory files with timestamps older than 3 months (check file modification
  date) — may contain outdated information about the project
- Memories referencing files, functions, or APIs that no longer exist

### Orphan files
- Memory files present in the directory but not listed in `MEMORY.md`
- Entries in `MEMORY.md` pointing to files that no longer exist
- Project memory directories where the associated project directory no
  longer exists on disk

### Index consistency
- `MEMORY.md` exceeding the load limit — only the first 200 lines or 25 KB,
  whichever comes first, are loaded into context
- Duplicate entries in `MEMORY.md` pointing to the same file

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

### Disabling custom MCP servers

Claude Desktop does NOT support toggling individual custom MCP servers
in `claude_desktop_config.json`. All entries in the `mcpServers` object
are loaded at startup regardless of naming. Renaming a key (e.g., adding
an underscore prefix) does NOT disable the server — it just creates a
server with a different name.

Options for occasional-use servers:
- **Leave enabled**: if RAM overhead is acceptable (~30-50 MB per server),
  keep the server running. It will function automatically when its backing
  service (e.g., Ollama, Docker) is available.
- **Remove and re-add**: delete the entry from the JSON when not needed,
  paste it back when needed. Keep a backup of the entry. This is the only
  reliable way to fully disable a custom server.

Account connectors (unlike config-file servers) can be toggled per
conversation from the "+" menu in the chat interface — a manual
mechanism, not an automatic one, but useful for occasional-use
connectors. Note also that where deferred tool loading is active
(Cowork), unused servers cost only their tool names in context, not the
full schemas: removing a server from the config saves RAM and startup
time, but fewer tokens than the tool count suggests.

## Context Cost Reference

RAM measures what a server costs the machine; context tokens measure what
the configuration costs every single conversation. On Chat and Cowork,
where the context window is the scarce resource, this cost usually
matters more than RAM.

### What consumes context

| Element | What gets injected | Rough cost |
|---------|-------------------|------------|
| MCP server tool | one schema per tool: name, description, parameters | ~150-600 tokens per tool |
| MCP server instructions | optional server-level instructions block | ~100-1,000 tokens per server |
| Installed skill | its description in the available-skills list | ~50-150 tokens per skill |
| Invoked skill | the full SKILL.md body, only when triggered | file characters ÷ 4 |
| Plugin | descriptions of every bundled skill | scales with bundle size |

### How to measure

Measure, don't multiply. The auditing session already contains the real
text of every loaded tool schema and skill description, so the cost is
computed from what is actually injected, not from a flat per-tool average.

- **Loaded tool schemas**: for each server whose tools are visible in the
  session, sum the characters of its tool definitions (name, description,
  parameters) and divide by 4. This is a per-server measurement.
- **Deferred tools**: if the platform lists a tool by name only, its schema
  is not injected until loaded — count only the name entry as always-paid
  cost, and note the on-demand cost separately.
- **Skill descriptions**: same method, characters of each description in
  the available-skills list ÷ 4.
- **Servers not loaded in the auditing session** (e.g., auditing a Desktop
  config from Claude Code): their schemas are not observable without
  launching the server, which the audit never does. Report them as "not
  measurable in this session" with the tool count if known. If a rough
  figure is indispensable, use a declared range (150-600 tokens per tool)
  and label it as an assumption, never as a measurement.
- In the report, state per server which method was used: measured,
  deferred, or assumed.
- Compare the total against the model's context window: a configuration
  consuming 15-20k tokens before the first user message deserves attention.

### Deferred loading changes the math

Some environments (Cowork, recent Claude Code versions) defer tool
schemas: only tool names appear in context until a tool is actually
loaded on demand. Where deferred loading is active, a high tool count
costs little until used, and the optimization target shifts to what is
always injected in full: skill descriptions, server instruction blocks,
and system prompt layers. State in the report whether the platform defers
schemas, because the same configuration can be cheap on one platform and
expensive on another.

### The whole system prompt, for proportion

The session's system prompt is larger than the user-managed layer alone:
above it sits the system-managed part — product identity and behavior
policies, tone defaults, usage instructions and schemas of built-in
tools, runtime metadata — defined by the platform and not configurable.
Estimate both parts from what is visible in the session context
(characters ÷ 4, ÷ 3 for Italian-heavy blocks) and report the split at
the top of the report. The system part is fixed overhead: report it for
proportion, never as something to optimize. Skill descriptions and tool
definitions are fully user-managed, exactly like the instruction files:
the user decides which servers, connectors, plugins and skills stay
loaded, and making that count visible is one of the audit's purposes.
Report a user-managed total that includes both instructions and
installed components. (A tool definition, or schema, is the
machine-readable card of a single tool — name, description, parameters —
injected into context by every server or connector, including those
bundled in plugins.)

### What to flag

- Servers exposing 15+ tools that the user touches rarely
- Multiple servers with overlapping tools (the duplicate pays twice)
- Plugins bundling many skills of which the user uses one or two — every
  bundled skill description is injected in every session
- Skill descriptions much longer than needed for reliable triggering: the
  description, not the body, is the always-paid cost
