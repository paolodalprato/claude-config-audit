---
name: claude-config-audit
description: >
  Comprehensive audit and optimization of Claude configuration: MCP servers,
  plugins, skills, and system prompt (CLAUDE.md + User Preferences). Detects
  duplicates, unused elements, resource waste, instruction conflicts, and
  overlapping capabilities. Produces a detailed report and interactively
  guides cleanup. Use this skill when the user asks to "audit my setup",
  "check my configuration", "optimize my Claude config", "clean up my MCP
  servers", "find duplicates in my plugins", "review my CLAUDE.md", or
  mentions wanting to improve their Claude Desktop, Cowork, or Claude Code
  setup. Also trigger when the user says things like "my setup feels bloated",
  "I have too many MCP servers", "which plugins should I keep", or "is my
  configuration efficient".
---

# Claude Config Audit

You are performing a comprehensive audit of the user's Claude configuration.
Your goal is to identify inefficiencies, duplicates, conflicts, and unused
elements across all configuration layers, then guide the user through an
interactive cleanup process.

## Environment Detection

Before starting, determine what tools you have available. This determines
how much you can do autonomously vs. how much you need from the user.

**Key platform constraint**: claude.ai does NOT have filesystem access
and cannot connect filesystem-related MCP servers. It works only with
session context (deferred tools, available skills) and whatever the user
provides in the prompt. Claude Desktop (Chat and Cowork) CAN have
filesystem access if the user has configured tools like Desktop Commander,
filesystem MCP, or similar. Claude Code has direct file access via its
built-in Read/Write/Edit tools.

**Full filesystem access** (Desktop Commander, filesystem MCP, or Claude
Code with direct file access — available in Claude Desktop and Claude Code,
never in claude.ai):
- Read configuration files directly from the user's machine
- List and analyze local skill directories
- Inspect MCP server configurations
- Apply changes with backup
- Detect OS from file paths and adapt commands accordingly

**Session context only** (no filesystem tool connected — always the case
in claude.ai, and in Claude Desktop/Cowork when no filesystem tool is
configured):
- MCP servers and skills are visible in the session context (deferred
  tools list, available skills list)
- Cannot read local config files — ask the user to paste or upload them
- Can still analyze everything the user provides and produce a full report
- Manual actions: provide exact file paths and content for the user to
  apply changes themselves

**Detecting available tools**: Check for the presence of Desktop Commander
(`mcp__Desktop_Commander__*`), filesystem MCP (`mcp__filesystem__*`),
Windows MCP (`mcp__Windows-MCP__*`), or similar file-access tools in the
session. In Claude Code, the Read/Write/Edit tools provide direct access.
If none are available, fall back to asking the user.

Adapt your workflow to the available tools. Never claim you can't help
just because you lack filesystem access — work with what you have.

## Audit Workflow

### Phase 1: Data Collection

Gather configuration data from all available sources. Collect as much as
possible in parallel to save time.

**Configuration files to locate** (paths vary by OS):

| File | Windows | macOS | Linux |
|------|---------|-------|-------|
| MCP config | `%APPDATA%\Claude\claude_desktop_config.json` | `~/Library/Application Support/Claude/claude_desktop_config.json` | `~/.config/Claude/claude_desktop_config.json` |
| Settings | `~\.claude\settings.json` | `~/.claude/settings.json` | `~/.claude/settings.json` |
| Global CLAUDE.md | `~\.claude\CLAUDE.md` | `~/.claude/CLAUDE.md` | `~/.claude/CLAUDE.md` |
| Local skills | `~\.claude\skills\` | `~/.claude/skills/` | `~/.claude/skills/` |
| Installed plugins | `~\.claude\plugins\installed_plugins.json` | `~/.claude/plugins/installed_plugins.json` | `~/.claude/plugins/installed_plugins.json` |

**Session context to analyze** (always available in Cowork/Chat/Code):
- Deferred tools list → reveals all active MCP servers and connectors
- Available skills list → reveals all loaded skills with their sources
- System prompt content → reveals User Preferences and CLAUDE.md as injected

**What to collect for each layer:**

1. **MCP Servers**: name, command, args, env vars (redact secrets), whether
   local install or npx/managed connector
2. **Plugins**: name, source marketplace, version, enabled/disabled status
3. **Skills**: name, location (local folder vs. plugin vs. native), structure
   (proper SKILL.md + reference/ or loose files)
4. **System Prompt**: User Preferences content, CLAUDE.md content, any
   project-level CLAUDE.md files

### Phase 2: Analysis

Run these checks against the collected data. Read
`reference/analysis-checks.md` for the detailed detection patterns.

**Cross-layer duplicate detection:**
- MCP servers providing the same functionality (e.g., two reasoning servers,
  two search APIs, two PDF tools)
- Skills available from multiple sources (local folder + plugin + remote plugin)
- Plugins installed from multiple marketplaces with the same name
- Instructions duplicated between User Preferences and CLAUDE.md

**Unused element detection:**
- MCP servers that require external services not running (Docker, Ollama, GIMP)
- Skills with no evidence of use (no output files, never mentioned in history)
- Setup/onboarding plugins still active after initial configuration
- API-based services with expired credentials or exhausted quotas

**Resource impact assessment:**
- Estimate RAM per MCP server (Node ~40-50MB, Python ~25-40MB, Docker ~200-500MB)
- Count total processes spawned at session start
- Identify servers using npx (adds startup latency for package download)
- Flag Docker-based servers as highest resource cost

**System prompt analysis:**
- Measure overlap between User Preferences and CLAUDE.md (identify paragraphs
  with >70% similarity)
- Check for contradictory instructions across levels
- Estimate token count for each prompt layer
- Identify instructions referencing removed/unavailable tools

**Project prompt analysis** (claude.ai — especially for free-tier users):
- Check for instruction overlap across Project system prompts (same rules
  repeated in multiple projects)
- Identify instructions that should be global (set once in a "General"
  project or in custom instructions) vs. project-specific
- Evaluate prompt quality: vague instructions, overly rigid formatting
  rules, missing context about audience/purpose
- Check for contradictions between projects (e.g., different tone
  instructions in different projects)
- Assess whether the project structure itself makes sense (too many
  projects with overlapping scope, or too few with overloaded prompts)

**Skill ecosystem analysis:**
- Identify orphan files in skill directories (.zip archives, backup folders,
  symlinks, non-skill files)
- Check if skills reference tools/MCP servers that are no longer available
- Detect skills that have been superseded by newer versions
- Flag generic plugin skills that don't match the user's work profile

### Phase 3: Report

Produce a structured report. Read `reference/report-template.md` for the
format. The report should include:

1. **Current state summary** — counts for each layer (MCP, plugins, skills)
2. **Issues found** — grouped by severity (duplicates, unused, conflicts,
   resource concerns)
3. **Recommended actions** — each with clear rationale and expected impact
4. **Questions for the user** — anything that requires their input to decide

Present the report to the user and pause. Don't proceed to changes until
they've reviewed and discussed.

### Phase 4: Interactive Validation

This phase is critical. Don't assume the analysis is complete — the user
knows their actual usage patterns better than any automated check.

For each recommended action, ask the user before proceeding. Group related
questions to avoid excessive back-and-forth, but don't batch more than 3-4
decisions at once.

Common discoveries during validation:
- "I don't use that MCP" → confirm removal
- "Actually I use that every week" → keep it, update the report
- "I installed that but never used it" → confirm removal
- "I'm not sure what that does" → explain it, then let them decide
- "That skill is connected to a project we'll resume soon" → keep it

Track all decisions. Update the action list based on user responses.

### Phase 5: Apply Changes

Only after the user has approved specific actions. Always in this order:

1. **Create backups** of every file you'll modify. Store in a timestamped
   backup directory (e.g., `~/.claude/backups/audit-YYYY-MM-DD/`).
2. **Apply changes one category at a time**, announcing each before doing it:
   - MCP servers (edit claude_desktop_config.json)
   - Plugins (edit settings.json)
   - System prompt (edit CLAUDE.md)
   - Skill folder (delete/move files)
3. **Verify each change** — parse JSON after editing to confirm validity,
   list directories after cleanup to confirm state.
4. **Document manual actions** — list anything you can't do that the user
   must do themselves (e.g., Cowork "Personalizza" menu changes, User
   Preferences text replacement).

### Phase 6: Final Report

Produce a final report documenting:
- Every change made (with before/after counts)
- Every manual action the user still needs to take (with exact instructions)
- Impact summary (RAM saved, tokens saved, duplicates eliminated)
- Backup location and restore instructions
- Anything that should be re-evaluated periodically

Save the report as a markdown file the user can keep for reference.

## Important Guidelines

**Be conservative with removals.** When in doubt, ask. A removed MCP server
that turns out to be needed is more disruptive than keeping one extra server
running. Prefer disabling over deleting when the option exists (e.g., set
plugin to `false` rather than removing the entry).

**Respect the user's workflow.** The audit should optimize for how the user
actually works, not for theoretical perfection. If they use Chat and Cowork
90% of the time, a CLI-only tool matters less. If they're a developer,
code-oriented plugins make sense even if used occasionally.

**Redact secrets.** When displaying configuration, always redact API keys,
passwords, and tokens. Show only enough to identify the service
(e.g., `"API_KEY": "E47F...1165"`).

**Explain the why.** For every recommendation, explain the reasoning.
"Remove X because it duplicates Y" is better than just "Remove X". The user
should understand each change well enough to make an informed decision.

**Platform awareness.** File paths, shell commands, and available tools
differ between Windows, macOS, and Linux. Detect the OS from context
(path separators, available commands, env vars) and adapt accordingly.
