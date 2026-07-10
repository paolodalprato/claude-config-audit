# claude-config-audit

A Claude skill that performs comprehensive audits of your Claude Desktop configuration across its three environments — Chat, Cowork, and Code: MCP servers, plugins, skills, system prompt layers, hooks, permissions, and memory files.

**Current version: 2.2.0** — see the [changelog](CHANGELOG.md).

## What it does

This skill analyzes your entire Claude setup across all configuration layers and identifies:

- **Duplicate MCP servers** — two reasoning tools, two search APIs, overlapping filesystem servers, multiple PDF tools
- **Unused elements** — servers requiring services not running, skills never used, setup plugins still active
- **Resource waste** — RAM per MCP server, startup latency from npx-based installs, Docker overhead
- **Broken servers** — read-only health checks: missing script paths, unresolvable npx packages, placeholder credentials, backing services (Docker, Ollama) not running
- **Context cost** — tokens injected in every session by MCP tool schemas, skill descriptions, and server instruction blocks, with deferred-loading awareness
- **Intervention plan** — every recommendation consolidated in one cross-layer table with explanation, difficulty, and expected impact, quick wins first
- **Interactive HTML report** (optional) — a self-contained tabbed page, one tab per environment plus the intervention plan, sortable tables, no external dependencies
- **System prompt bloat** — instructions duplicated between User Preferences and CLAUDE.md, stale references to removed tools
- **Instruction hierarchy** — the effective stack each environment receives, rules placed at the wrong level, drift in the shared core between environment configs, declared compensations, coverage gaps, and an override map separating real conflicts from legitimate specialization
- **Project prompt overlap** — same rules repeated across multiple Projects
- **Skill ecosystem issues** — orphan files, superseded versions, plugin-vs-local-skill duplicates
- **Plugin duplicates** — same plugin from multiple marketplaces, installed-but-never-enabled plugins, two data sources cross-referenced (`enabledPlugins` + `installed_plugins.json`)
- **Hooks and permissions issues** (Claude Code) — duplicate or conflicting hooks, overly broad permissions, stale rules referencing removed tools
- **Memory file hygiene** (Claude Code) — stale memories, orphan files, index inconsistencies
- **Per-environment analysis** — each environment audited as the complete stack it receives; duplicates and context cost judged where they are paid: a duplicate inside one environment is a finding, the same component in two environments usually is not

It then guides you through an interactive cleanup, creating backups before any change.

## How it works

The audit follows a 6-phase workflow:

1. **Data Collection** — reads config files and session context. Adapts to what's available (full filesystem access, session context only, or user-provided data)
2. **Analysis** — verifies each component once, then analyzes each environment (Chat, Cowork, Code) as the complete stack it receives: internal duplicates, measured context cost, unused elements, plus cross-environment checks and read-only health checks on MCP servers
3. **Report** — structured markdown report with a component inventory, findings per environment, resource impact summary, and a prioritized intervention plan; on request, also an interactive tabbed HTML version. Reports are written in the language you use with Claude
4. **Interactive Validation** — asks you about each recommendation before acting (3-4 decisions at a time)
5. **Apply Changes** — backups first, then changes one category at a time with JSON validation after each edit
6. **Final Report** — documents everything done, manual actions remaining, restore instructions, and periodic maintenance schedule

## The Three Environments

The audit covers Claude Desktop and treats each of its environments as a complete stack — what a session there actually receives:

| Layer | Chat | Cowork | Code |
|-------|------|--------|------|
| System prompt | User Preferences + project prompt | User Preferences + global CLAUDE.md + project instructions | CLAUDE.md (global + project-level), no User Preferences |
| MCP servers | claude_desktop_config.json | claude_desktop_config.json + connectors | ~/.claude.json (user scope) + .mcp.json (project) + claude_desktop_config.json |
| Skills | Account-level skills | Local + plugin skills | Local + plugin skills |
| Plugins | — | Marketplace plugins | Marketplace plugins (settings.json) |
| Hooks / Permissions | — | — | settings.json |
| Memory files | — | — | ~/.claude/projects/*/memory/ |

Components are shared across environments, so each is verified once — but duplicates, context cost, and coherence are judged per environment, where they are paid.

## Configuration files analyzed

| File | Description |
|------|-------------|
| `claude_desktop_config.json` | Desktop/Cowork MCP server definitions |
| `~/.claude.json` | Code user-scope MCP servers (`mcpServers` key) |
| `~/.claude/settings.json` | Code settings: hooks, permissions, enabled plugins, project-MCP approvals |
| `~/.claude/CLAUDE.md` | Global system prompt |
| `~/.claude/skills/` | Local skill directories |
| `~/.claude/plugins/installed_plugins.json` | Plugin installation metadata |
| `<project>/.mcp.json` | Project-level MCP servers (Code) |
| `<project>/.claude/settings.json` | Project-level settings (Code) |
| `<project>/CLAUDE.md` | Project-level system prompt |

## Installation

This skill is a folder you copy into your Claude skills directory. No dependencies, no build step, no terminal required.

### Option 1: Download as ZIP (no Git needed)

1. On this page, click the green **Code** button near the top right.
2. Select **Download ZIP**.
3. Extract the downloaded archive. You'll get a folder named `claude-config-audit-main` (or similar).
4. Rename it to `claude-config-audit` (remove the `-main` suffix).
5. Copy the entire folder into your Claude skills directory:
   - **Windows**: `C:\Users\<your username>\.claude\skills\`
   - **macOS**: `~/.claude/skills/`
   - **Linux**: `~/.claude/skills/`
6. If the `skills` folder doesn't exist yet, create it.

The final result should look like this:

```
~/.claude/skills/claude-config-audit/
├── SKILL.md
└── reference/
    ├── analysis-checks.md
    ├── health-checks.md
    ├── hierarchy-checks.md
    ├── report-html-template.html
    └── report-template.md
```

That's it. The skill will be available in your next Claude session.

### Option 2: Clone with Git

If you're familiar with Git, clone directly into the skills directory:

```bash
# macOS / Linux
cd ~/.claude/skills && git clone https://github.com/paolodalprato/claude-config-audit

# Windows
cd %USERPROFILE%\.claude\skills && git clone https://github.com/paolodalprato/claude-config-audit
```

The advantage of cloning is that future updates are a single `git pull` away.

### Verify the installation

Start a new Claude session and ask something like:

- "Audit my Claude configuration"
- "Check my setup for duplicates"
- "Is my configuration efficient?"
- "My setup feels bloated"

Claude should recognize the skill and begin the audit workflow.

## Structure

```
claude-config-audit/
├── SKILL.md                         # Main skill file (workflow + guidelines)
└── reference/
    ├── analysis-checks.md           # Detection patterns for all issue types
    ├── health-checks.md             # Read-only operational checks for MCP servers
    ├── hierarchy-checks.md          # Instruction hierarchy: placement, drift, specialization
    ├── report-html-template.html    # Self-contained tabbed HTML report (optional output)
    └── report-template.md           # Templates for initial and final reports
```

## What it checks

### MCP Servers
Known overlap patterns for reasoning servers, web search, filesystem access, PDF tools, and image generation. RAM estimates per server type (Node.js, Python, Docker). MCP server toggle limitations documented.

### Server health
Read-only verification that each server can actually run: script paths, runtimes, package resolvability, env vars, backing services. Servers are classified OK, degraded, broken, misconfigured, or unverifiable — the audit never launches a server to test it.

### Context cost
Token cost measured from the actual tool schemas, skill descriptions, and server instruction blocks visible in the auditing session — not from flat per-tool averages. Servers not loaded in the session are reported with declared assumptions. Accounts for platforms with deferred tool loading, where the math changes.

### Plugins
Same plugin from multiple marketplaces. Plugin vs. local skill overlap. Setup/onboarding plugins still active. Two data sources cross-referenced for completeness.

### Skills
Directory structure validation. Orphan files (archives, backups). Superseded skill versions (v2, agentic, enhanced).

### System Prompt
Overlap detection between User Preferences and CLAUDE.md. Stale references to removed tools. Token estimation per prompt layer.

### Instruction hierarchy
Effective stack per environment (what each session actually receives, including the fact that the Code environment never sees User Preferences). Rules placed at the wrong level. The unit of comparison is the stack, not the file: differences that compensate a missing level are declared compensation, not drift; drift is divergence in the shared core. A rewrite test ("in general X, but in this context Y") separates true conflicts from legitimate specialization, which goes to an informational override map instead of the issues list.

### Hooks & Permissions (Claude Code)
Duplicate or conflicting hooks. Broken hooks referencing missing scripts. Overly broad permission rules. Stale permissions for removed MCP servers.

### Memory Files (Claude Code)
Stale memories (3+ months old). Orphan files not in MEMORY.md index. Index consistency checks. Project memory directories for deleted projects.

## Contributing

Issues and pull requests are welcome. If you discover new MCP server overlap patterns, plugin conflicts, or platform-specific behaviors, feel free to contribute them to `reference/analysis-checks.md`; new operational verification patterns belong in `reference/health-checks.md`.

## License

MIT — see [LICENSE](LICENSE).
