# claude-config-audit

A Claude skill that performs comprehensive audits of your Claude configuration — MCP servers, plugins, skills, and system prompt (CLAUDE.md + User Preferences).

## What it does

This skill analyzes your entire Claude setup and identifies:

- **Duplicate MCP servers** — two reasoning tools, two search APIs, overlapping filesystem servers
- **Unused elements** — servers requiring services not running, skills never used, setup plugins still active
- **Resource waste** — RAM per MCP server, startup latency from npx-based installs, Docker overhead
- **System prompt bloat** — instructions duplicated between User Preferences and CLAUDE.md, stale references to removed tools
- **Project prompt overlap** (claude.ai) — same rules repeated across multiple Projects
- **Skill ecosystem issues** — orphan files, superseded versions, plugin-vs-local-skill duplicates

It then guides you through an interactive cleanup, creating backups before any change.

## How it works

The audit follows a 6-phase workflow:

1. **Data Collection** — reads config files (or asks you to paste them if no filesystem access)
2. **Analysis** — cross-references MCP, plugins, skills, and system prompt for issues
3. **Report** — structured markdown report with findings, recommendations, and resource impact
4. **Interactive Validation** — asks you about each recommendation before acting
5. **Apply Changes** — backups first, then changes one category at a time
6. **Final Report** — documents everything done, manual actions remaining, and restore instructions

## Compatibility

| Environment | Filesystem access | How it works |
|-------------|-------------------|--------------|
| **Claude Desktop (Chat)** | With Desktop Commander or filesystem MCP | Full autonomous audit |
| **Claude Desktop (Chat)** | Without filesystem tools | Analyzes session context + user-provided data |
| **Cowork** | With Desktop Commander or filesystem MCP | Full autonomous audit |
| **Cowork** | Without filesystem tools | Analyzes session context + user-provided data |
| **Claude Code** | Always (built-in Read/Write/Edit) | Full autonomous audit |
| **claude.ai** | Never | Analyzes session context + user-provided data |

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
- "Clean up my plugins"

## Structure

```
claude-config-audit/
├── SKILL.md                         # Main skill file
└── reference/
    ├── analysis-checks.md           # Detection patterns for duplicates and issues
    └── report-template.md           # Templates for audit and final reports
```

## License

MIT — see [LICENSE](LICENSE).
