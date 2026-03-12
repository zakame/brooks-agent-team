---
description: Directly spawn a Brooks Surgical Team using Claude Code Agent Teams — one independent session per role, working in parallel with a shared task list. Skips orientation and goes straight to spawning.
---

# Assemble Team with Agent Teams

When invoked, do the following steps in order.

## Step 1: Check Prerequisites

Check whether Agent Teams is enabled by looking for `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
in the environment or `settings.json`.

If it is NOT enabled, stop and tell the user:

> Agent Teams is not enabled. Add the following to your `~/.claude/settings.json` and restart:
>
> ```json
> {
>   "env": {
>     "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
>   }
> }
> ```
>
> Then re-run `/assemble-with-agent-teams`. Alternatively, run `/assemble-team` to use
> the single-session surgical team instead.

## Step 2: Quick Project Survey

Read just enough to know what work is ahead:
- Project name and primary language/framework
- Any open tasks, in-progress work, or recent git activity (`git log --oneline -5`)
- Presence of `CLAUDE.md`, `docs/`, `tests/` directories

Do NOT do a full orientation — that is `/assemble-team`'s job. This command is about
getting to work fast.

## Step 3: Determine Which Roles to Spawn

Ask the user one focused question:

> I'll spawn the core surgical team now. The default is **Surgeon (you) + Copilot + Tester**.
>
> Do you want any additional teammates?
> - **Editor** — for documentation-heavy work
> - **Toolsmith** — if you need new scripts or automation
> - **Language Lawyer** — if tricky framework/version edge cases are expected
> - **Program Clerk** — if the codebase needs reorganization alongside new work
>
> Reply with the roles to add, or say "default" to proceed with the core three.

Wait for the user's response before continuing.

## Step 4: Spawn the Team

Using the roles confirmed in Step 3, spawn the Agent Teams session with the following
instructions. Deliver the spawn message in this exact structure:

---

Spawn a surgical team for [PROJECT NAME] with the following teammates.
I am the **Surgeon** (team lead). Each teammate should invoke their assigned
`brooks-agent-team:` skill immediately at the start of their session and before
every task they claim.

### File ownership (to prevent conflicts)

[Assign based on your project survey — for example:]
- Copilot: read-only review access across all files; never edits source directly
- Tester: owns `tests/` and any `*.test.*` / `*.spec.*` files
- Editor: owns `docs/`, `README.md`, and any `*.md` documentation files
- Toolsmith: owns `scripts/`, `Makefile`, and any tooling/config files
- Language Lawyer: read-only research; communicates findings, does not edit files
- Program Clerk: proposes structure changes for Surgeon approval before executing

### Teammate instructions

**Copilot**
> You are the Copilot on this surgical team. At the start of this session and before
> every task you claim, invoke the `brooks-agent-team:copilot` skill.
> Your job: review completed work against its design intent. Claim tasks tagged
> `[review]`. Ask the Surgeon for spec or plan context before reviewing anything.
> Do not edit source code — report findings only.

**Tester**
> You are the Tester on this surgical team. At the start of this session and before
> every task you claim, invoke the `brooks-agent-team:tester` skill.
> Your job: adversarial test design and implementation. Claim tasks tagged `[test]`.
> Enumerate failure modes before writing any test code. Do not modify production code.

[Include the following only if user requested them:]

**Editor**
> You are the Editor on this surgical team. At the start of this session and before
> every task you claim, invoke the `brooks-agent-team:editor` skill.
> Your job: accurate, complete documentation. Claim tasks tagged `[docs]`.
> Read the code before writing about it. Never document what you haven't verified.

**Toolsmith**
> You are the Toolsmith on this surgical team. At the start of this session and before
> every task you claim, invoke the `brooks-agent-team:toolsmith` skill.
> Your job: build utilities that make the Surgeon permanently faster. Claim tasks
> tagged `[tooling]`. Every tool you build must have --help text and tests.

**Language Lawyer**
> You are the Language Lawyer on this surgical team. At the start of this session and
> before every task you claim, invoke the `brooks-agent-team:language-lawyer` skill.
> Your job: answer edge-case language and framework questions. Claim tasks tagged
> `[research]`. Cite the spec or docs — never guess. Say "I don't know" when uncertain.

**Program Clerk**
> You are the Program Clerk on this surgical team. At the start of this session and
> before every task you claim, invoke the `brooks-agent-team:program-clerk` skill.
> Your job: codebase organization and structure. Claim tasks tagged `[structure]`.
> Propose all reorganizations to the Surgeon for approval before executing.

### Initial shared task list

Create tasks based on the current project state observed in Step 2. Use these tags:
- `[implement]` — Surgeon claims these
- `[review]` — Copilot claims these (depend on corresponding `[implement]` tasks)
- `[test]` — Tester claims these (depend on corresponding `[implement]` tasks)
- `[docs]` — Editor claims these
- `[tooling]` — Toolsmith claims these
- `[research]` — Language Lawyer claims these
- `[structure]` — Program Clerk claims these

Add dependency links so `[review]` and `[test]` tasks are blocked until their
corresponding `[implement]` task is complete.

---

## Step 5: Confirm and Hand Off

After spawning, tell the user:

> Team is live. You are the Surgeon — claim `[implement]` tasks and your teammates
> will handle review, testing, and support in parallel.
>
> A few reminders:
> - Use **Shift+Down** to message a teammate directly
> - Two teammates must not edit the same file — follow the ownership assignments above
> - Run `/assemble-team` at any time for a full role reference
> - When the work is done, shut down teammates and clean up team resources
