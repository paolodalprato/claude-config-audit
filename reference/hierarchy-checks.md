# Instruction Hierarchy Analysis — Levels, Placement, Conflicts

Configuration instructions live at multiple levels: User Preferences,
global CLAUDE.md files, project prompts and project CLAUDE.md files.
The hierarchy exists to specialize: **a difference between levels is not
a defect, it is the system working as designed.** This file defines how
to tell apart what works as designed from what is misplaced, drifted,
or contradictory.

## Effective stack per platform

Before judging any rule, reconstruct what each session actually
receives. The same account produces different stacks on different
platforms:

| Session | Typical instruction stack |
|---------|---------------------------|
| claude.ai web | User Preferences + active Project prompt |
| Desktop Chat | User Preferences + active Project prompt |
| Cowork | User Preferences + Cowork global CLAUDE.md + project instructions |
| Code (CLI) | `~/.claude/CLAUDE.md` + project CLAUDE.md — no User Preferences |
| Code (Desktop app tab) | same instruction stack as the CLI |

Structural facts to surface in every audit:

- **Claude Code never sees User Preferences.** Any mandate living there
  simply does not exist in Code sessions. This is the most common source
  of coverage gaps (see below).
- **The two Code frontends share instruction files** (CLAUDE.md,
  settings, skills, plugins) **but diverge on MCP sources**: the
  Desktop-app Code tab also loads servers from
  `claude_desktop_config.json`, the standalone CLI does not (unless
  imported via `claude mcp add-from-claude-desktop`).
- Estimate tokens per level with the measured method (see Context Cost
  Reference in `analysis-checks.md`), and report the total per platform.

## The rewrite test

The discriminator between conflict and specialization. Take the general
rule and the specific rule and try to rewrite them as a single sentence:

> "In general X, but in this context Y."

If the rewrite is coherent and loses no meaning, the pair is a
**specialization** — the hierarchy doing its job. Example: global "Italian
as the default language" + project "English for this client, chat
included" rewrites cleanly and is not a conflict.

The test fails in two telling ways:

- **Same level, no scoping possible**: two rules in the same file (often
  added months apart) that contradict each other. No "context" can
  separate them — this is a contradiction, always a finding.
- **Copies, not scopes**: two global CLAUDE.md files on different
  platforms are not different levels of intent, they are accidental
  copies of the same level. Their differences are drift, not
  specialization (see below).

## Classification: four outcomes

### 1. Misplacement (finding)

A rule living at a level broader than its actual scope. Both conditions
must hold:

1. The rule fires only when a specific context is active — a named
   client, a single project, one workflow — not in every session.
2. A more specific home exists (a dedicated project, a project CLAUDE.md).

If only the first condition holds, the rule is a *conditional global
rule*: legitimate when no better level exists. If the better home cannot
be confirmed, degrade the finding to a question.

The cost argument: a misplaced rule pays its tokens in every session and
fires in almost none — the system-prompt equivalent of unused tool
schemas. The inverse also applies: the same rule repeated in several
project prompts, or at several simultaneously active levels, should be
promoted to one level and stated once.

### 2. Drift (finding)

Divergence between accidental copies of the same conceptual level —
typically the global CLAUDE.md of two platforms that started as the
same text. Compare rule by rule and report what each copy has that the
other lacks: diverged lists, rules present on one side only, same rule
phrased differently. None of it is specialization, because platforms are
not scopes of intent. Recommend either a primary copy with a tracked
derived copy, or an explicit decision to keep them separate with a
periodic consistency check.

### 3. Coverage gap (finding or question)

A rule present in one platform's stack and absent from another where it
would apply equally. Typical case: mandates living in User Preferences,
which Claude Code never receives. Not drift — nothing diverged — but a
whole level one platform does not get. Surface it and ask whether the
gap is intended; the fix is usually replicating the rule into the
platform's own level (e.g. `~/.claude/CLAUDE.md`).

### 4. Presumed specialization (not a finding)

Every cross-level difference that passes the rewrite test. It goes into
the override map (below) for awareness, never into the issues list.
Ambiguous cases — the rewrite works but the intent is unclear — become
questions in the interactive validation, not findings.

## Override map

Informational table for the report, not an issue list:

| General rule | Overridden where | By what | Declared? |
|--------------|------------------|---------|-----------|

"Declared" means the general rule itself anticipates the exception
("documentation in English, unless the project says otherwise"). A
declared override is the healthiest pattern in the whole hierarchy;
when a recurring override is undeclared, suggest declaring it in the
general rule.

## Ordering in the report

Report hierarchy results in this order: contradictions and coverage
gaps first (behavioral risk), then drift (silent divergence over time),
then misplacement (cost), then the override map (information only).
