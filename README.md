# claude-config-audit

A Claude skill that performs comprehensive audits of your Claude configuration — MCP servers, plugins, skills, system prompt, hooks, permissions, and memory files.

## What it does

This skill analyzes your entire Claude setup across all configuration layers and identifies:

- **Duplicate MCP servers** — two reasoning tools, two search APIs, overlapping filesystem servers, multiple PDF tools
- **Unused elements** — servers requiring services not running, skills never used, setup plugins still active
- **Resource waste** — RAM per MCP server, startup latency from npx-based installs, Docker overhead
- **System prompt bloat** — instructions duplicated between User Preferences and CLAUDE.md, stale references to removed tools
- **Project prompt overlap** (claude.ai) — same rules repeated across multiple Projects
- **Skill ecosystem issues** — orphan files, superseded versions, plugin-vs-local-skill duplicates
- **Plugin duplicates** — same plugin from multiple marketplaces, installed-but-never-enabled plugins, two data sources cross-referenced (`enabledPlugins` + `installed_plugins.json`)
- **Hooks and permissions issues** (Claude Code) — duplicate or conflicting hooks, overly broad permissions, stale rules referencing removed tools
- **Memory file hygiene** (Claude Code) — stale memories, orphan files, index inconsistencies
- **Cross-platform gaps** — detects if you have both Claude Desktop and Claude Code configured, and offers to audit both

It then guides you through an interactive cleanup, creating backups before any change.

## How it works

The audit follows a 6-phase workflow:

1. **Data Collection** — reads config files, session context, and cross-platform configs. Adapts to what's available (full filesystem access, session context only, or user-provided data)
2. **Analysis** — cross-references all layers (MCP, plugins, skills, system prompt, hooks, permissions, memory) for issues using documented detection patterns
3. **Report** — structured markdown report with platform info, findings per layer, resource impact summary, and recommendations
4. **Interactive Validation** — asks you about each recommendation before acting (3-4 decisions at a time)
5. **Apply Changes** — backups first, then changes one category at a time with JSON validation after each edit
6. **Final Report** — documents everything done, manual actions remaining, restore instructions, and periodic maintenance schedule

## Platform Compatibility

The skill works on all Claude platforms, adapting its scope to what each environment supports:

| Layer | claude.ai | Chat | Cowork | Code (CLI & Desktop app) |
|-------|-----------|------|--------|--------------------------|
| System prompt | Project prompts | CLAUDE.md | User Preferences + CLAUDE.md | CLAUDE.md (global + project-level) |
| MCP servers | Connectors (session context) | claude_desktop_config.json | claude_desktop_config.json | settings.json + .mcp.json |
| Skills | Project skills (session context) | Local + plugin skills | Local + plugin skills | Local + plugin skills |
| Plugins | — | — | Marketplace plugins | Marketplace plugins (settings.json) |
| Hooks / Permissions | — | — | — | settings.json |
| Memory files | — | — | — | ~/.claude/projects/*/memory/ |

All tiers (free and paid) have access to Projects, MCP connectors, and skills on claude.ai. The audit scope is the same regardless of plan.

### Cross-platform detection

When running with filesystem access, the skill checks whether the other platform is also configured:

- **From Claude Code**: checks if `claude_desktop_config.json` exists and offers to audit Desktop MCP servers too
- **From Claude Desktop**: checks if `settings.json` contains Code-specific keys (hooks, permissions) and offers to audit them

## Configuration files analyzed

| File | Description |
|------|-------------|
| `claude_desktop_config.json` | Desktop/Cowork MCP server definitions |
| `~/.claude/settings.json` | Code settings: MCP servers, hooks, permissions, enabled plugins |
| `~/.claude/CLAUDE.md` | Global system prompt |
| `~/.claude/skills/` | Local skill directories |
| `~/.claude/plugins/installed_plugins.json` | Plugin installation metadata |
| `<project>/.mcp.json` | Project-level MCP servers (Code) |
| `<project>/.claude/settings.json` | Project-level settings (Code) |
| `<project>/CLAUDE.md` | Project-level system prompt |

## Installation

### As a local skill (recommended)

Copy the entire `claude-config-audit` folder into your skills directory:

- **Windows**: `C:\Users\<username>\.claude\skills\`
- **macOS/Linux**: `~/.claude/skills/`

The skill will be automatically available in your next Claude session.

### Manual trigger

If the skill doesn't trigger automatically, you can invoke it with phrases like:

- "Audit my Claude configuration"
- "Check my MCP servers for duplicates"
- "Optimize my setup"
- "Is my configuration efficient?"
- "My setup feels bloated"
- "Which plugins should I keep?"

## Structure

```
claude-config-audit/
├── SKILL.md                         # Main skill file (workflow + guidelines)
└── reference/
    ├── analysis-checks.md           # Detection patterns for all issue types
    └── report-template.md           # Templates for initial and final reports
```

## What it checks

### MCP Servers
Known overlap patterns for reasoning servers, web search, filesystem access, PDF tools, and image generation. RAM estimates per server type (Node.js, Python, Docker). MCP server toggle limitations documented.

### Plugins
Same plugin from multiple marketplaces. Plugin vs. local skill overlap. Setup/onboarding plugins still active. Two data sources cross-referenced for completeness.

### Skills
Directory structure validation. Orphan files (archives, backups). Superseded skill versions (v2, agentic, enhanced).

### System Prompt
Overlap detection between User Preferences and CLAUDE.md. Stale references to removed tools. Token estimation per prompt layer.

### Hooks & Permissions (Claude Code)
Duplicate or conflicting hooks. Broken hooks referencing missing scripts. Overly broad permission rules. Stale permissions for removed MCP servers.

### Memory Files (Claude Code)
Stale memories (3+ months old). Orphan files not in MEMORY.md index. Index consistency checks. Project memory directories for deleted projects.

## Contributing

Issues and pull requests are welcome. If you discover new MCP server overlap patterns, plugin conflicts, or platform-specific behaviors, feel free to contribute them to `reference/analysis-checks.md`.

## License

MIT — see [LICENSE](LICENSE).
