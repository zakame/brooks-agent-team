---
description: Introduce the Brooks Surgical Team members, survey the current project, and present a contextual briefing on which roles are most relevant and how to invoke each one
---

# Assemble the Surgical Team

When invoked, do the following steps in order.

## Step 1: Orient

Invoke the `brooks-agent-team:using-brooks-team` skill to establish team framework context.

## Step 2: Survey the Project

Read the current project state:
- Check for `CLAUDE.md`, `README.md`, or any project documentation
- Look at the top-level directory structure
- Check `git log --oneline -10` if this is a git repository
- Identify the primary language and framework in use

If there is no project context (empty directory), skip to Step 3 and note that no project was found.

## Step 3: Present the Team Briefing

Present a project-contextual briefing in this format:

---

## Your Surgical Team for [Project Name or "New Project"]

**Your role: Surgeon (Chief Programmer)**
You own all critical design and implementation decisions. The team supports you.

---

**Copilot** — Code review and backup
- Invoke: `brooks-agent-team:copilot` skill (inline review), or dispatch `agents/copilot.md` (subagent review)
- Best for: [describe the main feature areas or recent work in this project where review would help]
- Trigger: After completing any feature of non-trivial size; before any merge

---

**Tester** — Adversarial test strategy
- Invoke: `brooks-agent-team:tester` skill, or dispatch `agents/tester.md`
- Best for: [describe any apparent testing gaps or recently implemented features]
- Trigger: Before calling any feature complete; when a bug escaped testing

---

**Administrator** — Task and project tracking
- Invoke: `brooks-agent-team:administrator` skill (inline only)
- Best for: [describe current project complexity — number of open tasks, known blockers]
- Trigger: Multi-task sessions; unclear priorities; scope creep

---

**Editor** — Documentation and clarity
- Invoke: `brooks-agent-team:editor` skill
- Best for: [describe current documentation state — README present/absent, API docs, etc.]
- Trigger: Before releases; when docs lag implementation; commit message review

---

**Program Clerk** — Code organization
- Invoke: `brooks-agent-team:program-clerk` skill (inline only)
- Best for: [describe any structural concerns observed — large files, unclear naming, etc.]
- Trigger: Files growing too large; inconsistent naming; tangled imports

---

**Toolsmith** — Custom tool builder
- Invoke: `brooks-agent-team:toolsmith` skill
- Best for: [describe any repetitive operations observed in the project]
- Trigger: Same manual operation performed 3+ times; workflows that take >5 minutes

---

**Language Lawyer** — Language and framework expertise
- Invoke: `brooks-agent-team:language-lawyer` skill
- Best for: [describe language/framework in use and any version-sensitive areas]
- Trigger: Edge cases; version differences; subtle API behavior; security-sensitive operations

---

## Step 4: Prompt for Next Action

End with:

> Which role would you like to activate first, or shall we proceed directly to the work?
>
> If you have a task in mind, describe it and I'll identify which team members should be involved.
