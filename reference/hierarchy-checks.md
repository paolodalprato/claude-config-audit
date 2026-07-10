# Instruction Hierarchy Analysis — Stacks, Placement, Compensation

Instructions live at multiple levels: User Preferences, global CLAUDE.md
files, project prompts and project CLAUDE.md files. Each Desktop
environment receives a different stack:

| Environment | Instruction stack |
|-------------|-------------------|
| Chat | User Preferences + active project prompt |
| Cowork | User Preferences + global CLAUDE.md (Cowork copy) + project instructions |
| Code | `~/.claude/CLAUDE.md` + project CLAUDE.md — no User Preferences |

Two principles govern the whole analysis:

- **The hierarchy exists to specialize.** A difference between levels is
  not a defect, it is the system working as designed.
- **The unit of comparison is the stack, not the file.** What must be
  equivalent across environments is the intent of the effective stack,
  not the text of individual files. Two global CLAUDE.md files serving
  different environments are not copies of one artifact.

Estimate tokens per level with the measured method (see Context Cost
Reference in `analysis-checks.md`) and report the total per environment.

## The rewrite test

The discriminator between conflict and specialization. Take the general
rule and the specific rule and try to rewrite them as a single sentence:

> "In general X, but in this context Y."

If the rewrite is coherent and loses no meaning, the pair is a
specialization — the hierarchy doing its job. Example: global "Italian
as the default language" + project "English for this client, chat
included" rewrites cleanly and is not a conflict.

The test fails in one telling way: two rules in the same file (often
added months apart) that contradict each other. No context can separate
them — a contradiction, always a finding.

## Within one stack: two outcomes

### Misplacement (finding)

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
schemas. The inverse also applies: the same rule repeated at several
simultaneously active levels should be promoted to one level and stated
once.

### Presumed specialization (not a finding)

Every cross-level difference that passes the rewrite test. It goes into
the override map (below) for awareness, never into the issues list.
Ambiguous cases — the rewrite works but the intent is unclear — become
questions in the interactive validation, not findings.

## Across stacks: three outcomes

### Declared compensation (expected difference, not a finding)

A rule that lives in User Preferences reaches Chat and Cowork but never
reaches Code, so the Code CLAUDE.md must restate it. The resulting
difference between the two CLAUDE.md files is the system being kept
coherent, not drift. Recommend making the compensation explicit — a
dedicated, marked section (e.g. "Method: shape-request") — so both
audits and human readers can tell the shared core from the compensation
layer at a glance.

Maintenance model to recommend: **shared core + marked compensation
block**. The environment files share an identical core; only the
environment that misses a level carries the compensation block. Core
changes propagate everywhere; compensation changes only where it lives.

### Drift (finding)

Divergence in the shared core *after subtracting declared compensations*:
same intent phrased differently on the two sides, rules present on one
side with no compensating reason, diverged lists. Compare rule by rule
and report what each side has that the other lacks.

### Coverage gap (finding or question)

A rule present in one environment's stack and absent from another where
it would apply equally — with no compensation in place. Surface it and
ask whether the gap is intended; the fix is usually adding the rule to
the missing environment's own level.

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
