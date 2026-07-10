# Report Template

Use this structure for both the initial audit report and the final report.
Adapt sections based on what was found — skip sections with no findings.

## Initial Audit Report

```markdown
# Claude Configuration Audit — [Date]

**Platform**: [detected platform: claude.ai / Chat / Cowork / Code]
**Platforms analyzed**: [list all platforms whose configs were inspected]

## 1. System Prompt

### Current state
- User Preferences: [line count] lines, ~[token estimate] tokens
- CLAUDE.md: [line count] lines, ~[token estimate] tokens
- Overlap: [percentage]% of content duplicated between the two

### Issues found
[List specific duplications, contradictions, stale references]

### Recommendation
[Consolidation strategy with rationale]

---

## 2. MCP Servers

### Current inventory ([count] servers)

| # | Server | Type | Install | Tools | Resource cost | Health |
|---|--------|------|---------|-------|---------------|--------|
[Table rows — Health from health-checks.md: OK / degraded / broken / misconfigured / unverifiable.
For context cost, mark per server whether the figure is measured, deferred, or assumed.]

### Issues found

#### Duplicates
[Group by functionality overlap]

#### Broken or misconfigured
[Servers failing health checks: missing paths, unresolvable packages,
placeholder credentials — with the exact failing check for each]

#### Unused/dormant
[Servers requiring unavailable external services]

#### Resource concerns
[High-cost servers: Docker, large models, etc.]

### Recommendations
[Numbered list with rationale and expected impact for each]

---

## 3. Plugins

### Current inventory ([enabled]/[total] enabled)

| # | Plugin | Source | Enabled | Status |
|---|--------|--------|---------|--------|
[Table rows]

### Issues found
[Duplicates, overlaps with skills, setup-only plugins]

### Recommendations
[Numbered list with rationale]

---

## 4. Skills

### Local skills ([count])
| # | Skill | Structure | Status |
|---|-------|-----------|--------|
[Table rows]

### Session skills ([count] from plugins/native)
[Summary of what's loaded and from where]

### Issues found
[Duplicates, orphan files, superseded versions, irrelevant plugin skills]

### Recommendations
[Numbered list with rationale]

---

## 5. Hooks & Permissions (Code only, skip if not applicable)

### Hooks ([count] configured)
| # | Event | Command | Scope | Status |
|---|-------|---------|-------|--------|
[Table rows]

### Permissions
| # | Rule | Type | Status |
|---|------|------|--------|
[Table rows]

### Issues found
[Duplicates, stale rules, overly broad permissions]

### Recommendations
[Numbered list with rationale]

---

## 6. Memory Files (skip if none exist)

### Current state
- Project memory directories: [count]
- Total memory files: [count]
- MEMORY.md indexes: [count]

### Issues found
[Stale files, orphans, index inconsistencies]

### Recommendations
[Numbered list with rationale]

---

## 7. Resource Impact Summary

| Metric | Current | After optimization | Savings |
|--------|---------|-------------------|---------|
| MCP servers | [n] | [n] | -[n] |
| MCP tools exposed | [n] | [n] | -[n] |
| Estimated RAM | [n] MB | [n] MB | -[n] MB |
| Estimated context cost | ~[n] tokens | ~[n] tokens | -[n]% |
| Enabled plugins | [n] | [n] | -[n] |
| Skill count (loaded) | [n] | [n] | -[n] |
| Hooks configured | [n] | [n] | -[n] |
| Memory files | [n] | [n] | -[n] |
| System prompt tokens | ~[n] | ~[n] | -[n]% |

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
