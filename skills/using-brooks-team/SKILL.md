---
name: using-brooks-team
description: Use at the start of any substantial development session to orient the Surgical Team framework and understand which roles apply to the current work
---

<SUBAGENT-STOP>
If you were dispatched as a Copilot, Tester, Editor, Toolsmith, Language Lawyer, Program Clerk, or Administrator subagent,
skip this skill. Your role instructions are in your agent prompt.
(Administrator is inline-only on Claude Code, Copilot CLI, OpenCode, and Grok Build; on Codex it is a dispatchable custom agent via `.codex/agents/administrator.toml`.)
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
| Administrator | `administrator` | Multi-task planning, tracking, prioritization | Yes (in Codex) / inline on other platforms |
| Editor | `editor` | Docs, specs, READMEs, commit messages | Optional dispatch in Codex; optional teammate via grok-team, hermes-team, or Claude Agent Teams |
| Program Clerk | `program-clerk` | File reorganization, naming, library structure | Optional dispatch in Codex; optional teammate via grok-team, hermes-team, or Claude Agent Teams |
| Toolsmith | `toolsmith` | Repetitive tasks, missing automation, workflow pain | Optional dispatch in Codex; optional teammate via grok-team, hermes-team, or Claude Agent Teams |
| Language Lawyer | `language-lawyer` | Framework subtlety, edge case, version concern | Yes (or inline) |

## When to Dispatch vs. Inline Guidance

```
Dispatch subagent for:             Inline guidance for:
──────────────────────             ────────────────────
Code review (Copilot)              Planning (Administrator)†
Test writing (Tester)              File organization (Program Clerk)†
Language investigations (Lawyer)   Small tool scripts (Toolsmith)
Large tool builds (Toolsmith)
Doc writing passes (Editor)
```

† On Codex, Administrator, Editor, Toolsmith, and Program Clerk are also dispatchable as custom agents under `.codex/agents/`.

## Skill Priority

1. **Surgeon** — always active during implementation
2. **Process roles** (Administrator, Program Clerk) — before starting or restructuring
3. **Quality roles** (Copilot, Tester) — after completing implementation units
4. **Support roles** (Editor, Toolsmith, Language Lawyer) — as specific needs arise

## The Core Rule

**Invoke a role's skill before acting in that role** — the skill carries the protocol the role follows, so acting without it means improvising the role.

For any user request: if it's implementation work, invoke the surgeon skill; if it's review/quality work, invoke copilot or tester; if it's planning/tracking, invoke administrator; for any other supporting need, invoke the relevant role skill — otherwise, proceed.

## User Instructions Always Win

Team skills guide HOW to work, not WHAT to build. If the user's CLAUDE.md or direct instructions conflict with a skill, follow the user. The team serves the project; the project serves the user.
