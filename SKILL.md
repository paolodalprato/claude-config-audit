---
name: claude-config-audit
version: 2.2.0
description: >
  Comprehensive audit and optimization of your Claude Desktop
  configuration across its three environments — Chat, Cowork, and Code:
  MCP servers, plugins, skills, system prompt layers (User Preferences,
  CLAUDE.md, project instructions), hooks, permissions, and memory
  files. Detects duplicates, unused elements, resource waste,
  instruction conflicts, and overlapping capabilities. Produces a
  detailed report and interactively guides cleanup. Use this skill when
  the user asks to "audit my setup", "check my configuration", "optimize
  my Claude config", "clean up my MCP servers", "find duplicates in my
  plugins", "review my CLAUDE.md", "audit my hooks", "review my
  permissions", "clean up my memory files", or mentions wanting to
  improve their Claude Desktop setup. Also trigger when the user says
  things like "my setup feels bloated", "I have too many MCP servers",
  "which plugins should I keep", or "is my configuration efficient".
---

# Claude Config Audit

You are performing a comprehensive audit of the user's Claude Desktop
configuration, covering its three environments: Chat, Cowork, and Code.
Your goal is to identify inefficiencies, duplicates, conflicts, and
unused elements — analyzing each environment as the complete stack it
actually receives — then guide the user through an interactive cleanup
process.

## Environment Detection

Before starting, determine what tools you have available. This determines
how much you can do autonomously vs. how much you need from the user.

**Full filesystem access** (Desktop Commander, filesystem MCP, or the
Code environment with its direct file tools):
- Read configuration files directly from the user's machine
- List and analyze local skill directories
- Inspect MCP server configurations
- Apply changes with backup
- Detect OS from file paths and adapt commands accordingly

**Session context only** (no filesystem tool connected):
- MCP servers and skills are still visible in the session context
  (deferred tools list, available skills list), and the injected system
  prompt reveals the instruction stack of the current environment
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

## The Three Desktop Environments

The audit covers Claude Desktop and treats each of its environments as
a complete stack — what a session there actually receives:

| Layer | Chat | Cowork | Code |
|-------|------|--------|------|
| System prompt | User Preferences + project prompt | User Preferences + global CLAUDE.md + project instructions | CLAUDE.md (global + project-level), no User Preferences |
| MCP servers | claude_desktop_config.json | claude_desktop_config.json + connectors | ~/.claude.json (user scope) + .mcp.json (project) + claude_desktop_config.json |
| Skills | Account-level skills | Local + plugin skills | Local + plugin skills |
| Plugins | — | Marketplace plugins | Marketplace plugins (settings.json) |
| Hooks / Permissions | — | — | settings.json |
| Memory files | — | — | ~/.claude/projects/*/memory/ |

Components are shared across environments (the same server definition
can feed more than one environment), so verify each component once, but
judge duplicates, context cost, and coherence per environment: a
duplicate inside one environment is paid in every session there, while
the same component appearing in two different environments is usually
legitimate.

## Audit Workflow

### Phase 1: Data Collection

Gather configuration data from all available sources. Collect as much as
possible in parallel to save time.

**Configuration files to locate** (paths vary by OS):

| File | Windows | macOS | Linux |
|------|---------|-------|-------|
| Desktop MCP config | `%APPDATA%\Claude\claude_desktop_config.json` | `~/Library/Application Support/Claude/claude_desktop_config.json` | `~/.config/Claude/claude_desktop_config.json` |
| Code user MCP config (`mcpServers`) | `~\.claude.json` | `~/.claude.json` | `~/.claude.json` |
| Code settings (hooks, permissions, plugins, project-MCP approvals) | `~\.claude\settings.json` | `~/.claude/settings.json` | `~/.claude/settings.json` |
| Global CLAUDE.md | `~\.claude\CLAUDE.md` | `~/.claude/CLAUDE.md` | `~/.claude/CLAUDE.md` |
| Local skills | `~\.claude\skills\` | `~/.claude/skills/` | `~/.claude/skills/` |
| Installed plugins (metadata) | `~\.claude\plugins\installed_plugins.json` | `~/.claude/plugins/installed_plugins.json` | `~/.claude/plugins/installed_plugins.json` |
| Project MCP (Code) | `<project>\.mcp.json` | `<project>/.mcp.json` | `<project>/.mcp.json` |
| Project settings (Code) | `<project>\.claude\settings.json` | `<project>/.claude/settings.json` | `<project>/.claude/settings.json` |
| Project CLAUDE.md | `<project>\CLAUDE.md` | `<project>/CLAUDE.md` | `<project>/CLAUDE.md` |

**Session context to analyze** (always available):
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
5. **Claude Code settings** (only in Code):
   - `mcpServers` in `~/.claude.json`: user-scope MCP servers, added via
     `claude mcp add`. These do NOT live in `settings.json` — an audit that
     skips `~/.claude.json` misses them entirely
   - `enabledPlugins` in `settings.json`: plugin enabled/disabled state
     (cross-reference with `installed_plugins.json` for metadata: version,
     install date, marketplace)
   - `enabledMcpjsonServers` in `settings.json`: MCP servers enabled from
     project `.mcp.json` files
   - `hooks` in `settings.json`: shell commands triggered on events
     (pre/post tool calls)
   - `permissions` in `settings.json`: allow/deny rules for tool execution

**Scope**: audit all three environments by default — the configuration
files of Chat, Cowork, and Code all live on the same machine. In the
report, always state which environments were analyzed so the user knows
whether the audit covers their full setup.

### Phase 2: Analysis

Run these checks against the collected data. Read
`reference/analysis-checks.md` for the detailed detection patterns,
`reference/health-checks.md` for the operational verification of MCP
servers, and `reference/hierarchy-checks.md` for the instruction
hierarchy analysis.

Structure the analysis in two passes. First the **component inventory**:
verify each component (config file, MCP server, skill, plugin) exactly
once, regardless of how many environments use it. Then the **environment
assembly**: for each of the three environments, reconstruct the stack it
receives and judge duplicates, context cost, and unused elements there —
that is where they are paid. Cross-environment checks (stack
equivalence, compensation, coverage gaps) come last.

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

**Operational health checks** (filesystem access required):
- Verify each MCP server can actually run: script paths exist, runtimes
  and packages resolve, env vars are set and not placeholders, backing
  services (Docker, Ollama) are reachable. Never launch the server itself.
- Classify each server: OK / degraded / broken / misconfigured /
  unverifiable. Patterns and OS commands in `reference/health-checks.md`.
- Feed the status into the MCP inventory table of the report.

**Context cost assessment:**
- Measure the tokens injected per session from what is actually visible
  in the session context: characters of tool schemas, skill descriptions,
  and server instruction blocks, divided by 4. Fall back to count-based
  ranges only for servers not loaded in the session, labeled as assumptions
- Measure the total system prompt of the auditing session and split it:
  system-managed part (product and tool instructions, tool schemas,
  runtime metadata — fixed, not auditable) vs. user-managed part
  (instructions and installed components — the audit's actionable
  surface). Report the split at the top of the report. The system parts
  of the other environments are not observable from this session: say
  so, don't guess them
- Note whether the platform defers tool schemas — it changes what to
  optimize (see Context Cost Reference in `reference/analysis-checks.md`)
- Flag high-tool-count servers that are rarely used, and plugins bundling
  many skills of which the user uses few

**System prompt analysis:**
- Measure overlap between User Preferences and CLAUDE.md (identify paragraphs
  with >70% similarity)
- Check for contradictory instructions across levels
- Estimate token count for each prompt layer
- Identify instructions referencing removed/unavailable tools

**Instruction hierarchy analysis** (cross-environment):
- Reconstruct the effective stack each environment receives: which
  levels, in what order, at what measured token cost — including the
  fact that the Code environment never sees User Preferences
- The unit of comparison is the stack, not the file: two global
  CLAUDE.md files serving different environments are not copies, and
  their expected differences (declared compensation for a missing
  level) are not drift. Taxonomy and tests in
  `reference/hierarchy-checks.md`
- Differences that pass the rewrite test ("in general X, but in this
  context Y") are specialization, not findings: they go to the override
  map, and ambiguous cases become validation questions

**Project prompt analysis**:
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

**Hooks and permissions analysis** (Claude Code only):
- Duplicate or conflicting hooks on the same event
- Hooks referencing scripts or paths that don't exist
- Overly broad permission rules (user awareness)
- Stale permissions referencing removed MCP server tools

**Memory files analysis** (Claude Code only):
- Stale memories (files not updated in 3+ months)
- Orphan files (not in MEMORY.md index, or index entries with missing files)
- Project memory directories for projects that no longer exist on disk
- MEMORY.md exceeding the load limit — only the first 200 lines or 25 KB,
  whichever comes first, are loaded into context

### Phase 3: Report

Produce a structured report. Read `reference/report-template.md` for the
format. The report should include:

1. **Component inventory** — every config file, MCP server, skill and
   plugin, each verified once, with health status
2. **One section per environment** (Chat, Cowork, Code) — the stack it
   receives, its context cost, its internal duplicates and unused elements
3. **Cross-environment checks** — stack equivalence, declared
   compensations, coverage gaps, override map
4. **Intervention plan** — one consolidated table: every action with its
   explanation, difficulty (low / medium / high) and expected impact,
   ordered so quick wins come first. Scale definitions in
   `reference/report-template.md`
5. **Questions for the user** — anything that requires their input to decide

After delivering the markdown report, offer an interactive HTML version:
a tabbed, self-contained page with one tab per layer plus the
intervention plan. If the user accepts, read
`reference/report-html-template.html` and populate its placeholders with
the report content, in the user's language. The file must remain fully
self-contained — inline CSS and JS only, no CDN or external resources —
so it opens offline months later. Save it next to the markdown report,
same base name, `.html` extension. The markdown report remains the
canonical output for archiving and comparing audits over time.

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
   - MCP servers (edit claude_desktop_config.json and/or settings.json)
   - Plugins (edit settings.json)
   - System prompt (edit CLAUDE.md)
   - Skill folder (delete/move files)
   - Hooks and permissions (edit settings.json — Code only)
   - Memory files (delete stale files, update MEMORY.md — Code only)
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
90% of the time, a Code-only tool matters less. If they're a developer,
code-oriented plugins make sense even if used occasionally.

**The report is a photograph.** Document the current state only. Never
compare with previous audits or reference earlier reports unless the
user explicitly asks for a comparison.

**Verify before classifying.** Before labeling a folder as leftover or
orphaned, open its SKILL.md (or main file) and read it. When reporting
the location of a finding, cite the exact file and JSON key you read,
never one reconstructed from memory.

**Write in the user's language.** All user-facing output — the markdown
report, the HTML report, validation questions, recommendations — is
written in the language the user is using in the conversation. The
templates are in English, but they define structure, not language:
translate headings, labels, and table headers. Keep file paths,
configuration keys, and technical identifiers untranslated.

**Redact secrets.** When displaying configuration, always redact API keys,
passwords, and tokens. Show only enough to identify the service
(e.g., `"API_KEY": "E47F...1165"`).

**Explain the why.** For every recommendation, explain the reasoning.
"Remove X because it duplicates Y" is better than just "Remove X". The user
should understand each change well enough to make an informed decision.

**Platform awareness.** File paths, shell commands, and available tools
differ between Windows, macOS, and Linux. Detect the OS from context
(path separators, available commands, env vars) and adapt accordingly.
