---
name: using-brooks-team
description: Use at the start of any substantial development session to orient the Surgical Team framework and understand which roles apply to the current work
---

<SUBAGENT-STOP>
If you were dispatched as a Copilot, Tester, Editor, Toolsmith, Language Lawyer, or Program Clerk subagent,
skip this skill. Your role instructions are in your agent prompt.
(Administrator is intentionally omitted — it has no dispatch template and is inline-only.)
</SUBAGENT-STOP>

# Using the Brooks Surgical Team

This framework organizes AI-assisted software development around Fred Brooks' Surgical Team model from *The Mythical Man-Month*. One chief programmer (the Surgeon) does all critical design and implementation work, supported by specialized roles that keep the Surgeon focused.

**You are the Surgeon.** All other roles exist to serve your ability to make good decisions and write good code.

## Team Role Map

| Role | Skill | When to Invoke | Subagent? |
|------|-------|----------------|-----------|
| Surgeon | `surgeon` | All implementation work | No — you ARE the surgeon |
| Copilot | `copilot` | Before completing any significant feature | Yes (or inline review) |
| Tester | `tester` | Any feature, bugfix, or quality concern | Yes (or inline) |
| Administrator | `administrator` | Multi-task planning, tracking, prioritization | No — inline only |
| Editor | `editor` | Docs, specs, READMEs, commit messages | Optional (via agent-teams) |
| Program Clerk | `program-clerk` | File reorganization, naming, library structure | Optional (via agent-teams) |
| Toolsmith | `toolsmith` | Repetitive tasks, missing automation, workflow pain | Optional (via agent-teams) |
| Language Lawyer | `language-lawyer` | Framework subtlety, edge case, version concern | Yes (or inline) |

## When to Dispatch vs. Inline Guidance

```
Dispatch subagent for:             Inline guidance for:
──────────────────────             ────────────────────
Code review (Copilot)              Planning (Administrator)
Test writing (Tester)              File organization (Program Clerk)
Language investigations (Lawyer)   Small tool scripts (Toolsmith)
Large tool builds (Toolsmith)
Doc writing passes (Editor)
```

## Skill Priority

1. **Surgeon** — always active during implementation
2. **Process roles** (Administrator, Program Clerk) — before starting or restructuring
3. **Quality roles** (Copilot, Tester) — after completing implementation units
4. **Support roles** (Editor, Toolsmith, Language Lawyer) — as specific needs arise

## The Core Rule

**Invoke the relevant role skill BEFORE acting in that role.** Even a 1% chance a role applies means invoking the skill to check. Supporting roles are not optional suggestions — they are mandatory process gates that protect the Surgeon's work.

```dot
digraph team_flow {
    "User request" [shape=doublecircle];
    "Is this implementation work?" [shape=diamond];
    "Invoke surgeon skill" [shape=box];
    "Is this review/quality work?" [shape=diamond];
    "Invoke copilot or tester skill" [shape=box];
    "Is this planning/tracking?" [shape=diamond];
    "Invoke administrator skill" [shape=box];
    "Other supporting need?" [shape=diamond];
    "Invoke relevant role skill" [shape=box];
    "Proceed" [shape=doublecircle];

    "User request" -> "Is this implementation work?";
    "Is this implementation work?" -> "Invoke surgeon skill" [label="yes"];
    "Is this implementation work?" -> "Is this review/quality work?" [label="no"];
    "Invoke surgeon skill" -> "Proceed";
    "Is this review/quality work?" -> "Invoke copilot or tester skill" [label="yes"];
    "Is this review/quality work?" -> "Is this planning/tracking?" [label="no"];
    "Invoke copilot or tester skill" -> "Proceed";
    "Is this planning/tracking?" -> "Invoke administrator skill" [label="yes"];
    "Is this planning/tracking?" -> "Other supporting need?" [label="no"];
    "Invoke administrator skill" -> "Proceed";
    "Other supporting need?" -> "Invoke relevant role skill" [label="yes"];
    "Other supporting need?" -> "Proceed" [label="no"];
    "Invoke relevant role skill" -> "Proceed";
}
```

## User Instructions Always Win

Team skills guide HOW to work, not WHAT to build. If the user's CLAUDE.md or direct instructions conflict with a skill, follow the user. The team serves the project; the project serves the user.
