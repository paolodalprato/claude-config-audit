# Report Template

Use this structure for both the initial audit report and the final
report. Adapt sections based on what was found — skip sections with no
findings.

The report is a photograph of the current state: no comparisons with
previous audits, no references to earlier reports, and no searching for
them — don't even ask. If the user wants a comparison, they will say so
and provide the earlier report.

The template is written in English but defines structure, not language:
produce the report in the language the user is using, translating
headings, labels, and table headers. Keep file paths, configuration
keys, and other technical identifiers as they are.

## Initial Audit Report

```markdown
# Claude Desktop Configuration Audit — [Date]

**Environments analyzed**: [Chat / Cowork / Code]

**System prompt footprint of this session** ([environment]):

| Part | ~Tokens | Auditable? |
|------|---------|------------|
| System-managed: product instructions, built-in tools, runtime metadata | | No — fixed overhead |
| User-managed, instructions: User Preferences, CLAUDE.md, project prompt | | Yes |
| User-managed, installed components: skill descriptions + tool definitions exposed by MCP servers and connectors (including those brought by plugins) + server instruction blocks | | Yes |
| **User-managed total (instructions + components)** | | **Yes — the audit's actionable surface** |
| Total | | |

[Both user-managed rows are equally under the user's control: the user
decides which servers, connectors, plugins and skills stay loaded, and
making that count visible is one of the audit's purposes — never present
the components as second-class. A "tool definition" (schema) is the
machine-readable card of a single tool — name, description, parameters —
that every server or connector injects into context so the model can
call it. Figures measured from the auditing session's context; the
system parts of the other environments are not observable from here —
say so instead of guessing.]

## 1. Component Inventory

[Each component verified once, regardless of how many environments use
it. Health from health-checks.md: OK / degraded / broken /
misconfigured / unverifiable.]

### Instruction files
| File / block | Level | Lines | ~Tokens |
|--------------|-------|-------|---------|
[User Preferences blocks, global CLAUDE.md files, project prompts]

### MCP servers
| # | Server | Defined in | Install | Tools | Health | Notes |
|---|--------|-----------|---------|-------|--------|-------|
[For context cost, mark per server whether the figure is measured,
deferred, or assumed.]

### Skills
| # | Skill | Location | Structure | Notes |
|---|-------|----------|-----------|-------|

### Plugins
| # | Plugin | Marketplace | Enabled | Notes |
|---|--------|-------------|---------|-------|

---

## 2. Environment: Chat

**Stack received**: User Preferences + active project prompt
**MCP servers loaded**: [from claude_desktop_config.json]
**Skills**: [account-level skills]

### Issues in this environment
[Duplicates inside this environment, context cost, unused elements]

---

## 3. Environment: Cowork

**Stack received**: User Preferences + global CLAUDE.md + project instructions
**MCP servers / connectors loaded**: [claude_desktop_config.json + connectors]
**Skills and plugins**: [local skills, plugin skills, marketplace plugins]
**Context cost**: [measured — typically the environment where bundles
and tool schemas weigh most]

### Issues in this environment
[Duplicates inside this environment, context cost, unused elements]

---

## 4. Environment: Code

**Stack received**: global CLAUDE.md + project CLAUDE.md (no User Preferences)
**MCP servers loaded**: [~/.claude.json + .mcp.json + claude_desktop_config.json]
**Hooks / permissions**: [from settings.json]
**Memory files**: [~/.claude/projects/*/memory/]

### Issues in this environment
[Duplicates inside this environment, unused plugins, hooks and
permissions concerns, memory hygiene]

---

## 5. Cross-Environment Checks

### Stack equivalence and compensation
[Differences that compensate a missing level are declared compensation,
not drift. Drift is divergence in the shared core after subtracting
compensations. See hierarchy-checks.md.]

### Coverage gaps
[Rules present in one environment's stack, absent where they would
apply equally, with no compensation in place]

### Override map
| General rule | Overridden where | Declared? |
|--------------|------------------|-----------|
[Informational, not issues: legitimate specializations across levels]

---

## 6. Resource Impact Summary

| Metric | Chat | Cowork | Code |
|--------|------|--------|------|
| MCP servers loaded | | | |
| MCP tools exposed | | | |
| Estimated RAM | | | |
| Context cost (state method per figure) | | | |
| Skills visible | | | |
| Plugins enabled | | | |

---

## 7. Intervention Plan

[Consolidated list of every recommended action, ordered by convenience:
quick wins — low difficulty, high impact — first.]

| # | Intervention | Where | Why | Difficulty | Expected impact | Your decision needed? |
|---|--------------|-------|-----|------------|-----------------|----------------------|
[Table rows]

**Difficulty scale**: low = single file edit or toggle, reversible in
seconds; medium = coordinated changes across files or environments;
high = requires investigation, migration, or restructuring before acting.

**Impact**: quantify with the audit's own measurements where available
(tokens per session, RAM, server and plugin counts); use qualitative
impact (clarity, safety, maintainability) where numbers don't apply.
Never present an assumed figure as a measured one.

---

## 8. Questions for You

[Numbered list of decisions requiring user input]
```

## Final Report

```markdown
# Configuration Audit — Final Report

[Date]

## 1. Changes Applied

### MCP Servers
**Before**: [n] servers → **After**: [n] servers

| Server removed | Reason |
|----------------|--------|
[Table rows]

| Server kept | Purpose |
|-------------|---------|
[Table rows]

### Plugins
**Before**: [n] enabled → **After**: [n] enabled

| Plugin disabled | Reason |
|-----------------|--------|
[Table rows]

| Plugin kept | Purpose |
|-------------|---------|
[Table rows]

### System Prompt (CLAUDE.md)
**Before**: ~[n] lines → **After**: ~[n] lines (-[n]%)

| Change | Detail |
|--------|--------|
[Table rows]

### Skill Directory
| Action | Element | Reason |
|--------|---------|--------|
[Table rows]

### Hooks & Permissions (Code only, skip if not applicable)
| Action | Element | Reason |
|--------|---------|--------|
[Table rows]

### Memory Files (skip if not applicable)
| Action | Element | Reason |
|--------|---------|--------|
[Table rows]

---

## 2. Manual Actions Required

[Numbered list with exact instructions for each action
the user must perform manually — e.g., changes in the
Cowork "Personalizza" menu, User Preferences text replacement]

---

## 3. Impact Summary

| Metric | Before | After | Change |
|--------|--------|-------|--------|
[Table with all metrics]

---

## 4. How to Restore

Backups location: `[path]`

[Restore instructions for each file]

---

## 5. Periodic Maintenance

Consider re-running this audit:
- After installing new MCP servers or plugins
- Every 2-3 months as a hygiene check
- After major Claude updates that add new native capabilities
  (which may make existing MCP servers redundant)
```
