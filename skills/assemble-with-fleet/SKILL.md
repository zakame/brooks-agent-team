---
name: assemble-with-fleet
description: "Copilot CLI fleet mode or OpenCode task tool — Spawn a Brooks Surgical Team using parallel subagents — one independent agent per role, working in parallel with a shared task list. Skips orientation and goes straight to spawning."
---

# Assemble Team with Fleet Mode

When invoked, do the following steps in order.

## Step 1: Quick Project Survey

Read just enough to know what work is ahead:
- Project name and primary language/framework
- Any open tasks, in-progress work, or recent git activity (`git log --oneline -5`)
- Presence of `AGENTS.md`, `COPILOT.md`, `docs/`, `tests/` directories

Do NOT do a full orientation — that is the `assemble-team` skill's job. This skill is
about getting to work fast.

## Step 2: Determine Which Roles to Spawn

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

## Step 3: Build the Shared Task List

Using the project survey from Step 1, construct an initial task list.

**Copilot CLI:** use the session SQL database. Insert tasks with appropriate tags:
- `[implement]` — Surgeon claims these
- `[review]` — Copilot claims these (depend on corresponding `[implement]` tasks)
- `[test]` — Tester claims these (depend on corresponding `[implement]` tasks)
- `[docs]` — Editor claims these (if Editor role was requested)
- `[tooling]` — Toolsmith claims these (if Toolsmith role was requested)
- `[research]` — Language Lawyer claims these (if Language Lawyer role was requested)
- `[structure]` — Program Clerk claims these (if Program Clerk role was requested)

Add dependency links so `[review]` and `[test]` tasks are blocked until their
corresponding `[implement]` task is complete.

**OpenCode:** do NOT rely on the session SQL database. Maintain the shared task list
in the main thread and include the current task list (with dependencies) in each
subagent prompt so they can claim work without database access.

## Step 4: Spawn the Team

Using the roles confirmed in Step 2, spawn each teammate directly via the `task` tool.

**Copilot CLI:** use `mode: "background"` and any fleet-specific fields required.
**OpenCode:** do not use `mode`. Always include `subagent_type`. If resuming an
existing background agent, include both `task_id` and `subagent_type` together.

Do NOT ask the user to copy-paste anything or run `/fleet` manually — spawn the agents
yourself.

**File ownership to include in each agent's prompt (assign based on your project survey):**
- Copilot: read-only review access across all files; never edits source directly
- Tester: owns `tests/` and any `*.test.*` / `*.spec.*` files
- Editor: owns `docs/`, `README.md`, and any `*.md` documentation files
- Toolsmith: owns `scripts/`, `Makefile`, and any tooling/config files
- Language Lawyer: read-only research; communicates findings, does not edit files
- Program Clerk: proposes structure changes for Surgeon approval before executing

### Copilot (always spawn)

Copilot CLI: use `agent_type: "copilot"` (this is a configured custom agent).
OpenCode: use `subagent_type: "copilot"`.

Prompt:

> You are the Copilot on the surgical team for [PROJECT NAME].
> Claim tasks tagged `[review]` from the shared task list in the session SQL database.
> Ask the Surgeon for spec or plan context before reviewing anything.
> Do not edit source code — report findings only.
> File ownership: read-only access across all files.

### Tester (always spawn)

Copilot CLI: use `agent_type: "tester"` (this is a configured custom agent).
OpenCode: use `subagent_type: "tester"`.

Prompt:

> You are the Tester on the surgical team for [PROJECT NAME].
> Claim tasks tagged `[test]` from the shared task list in the session SQL database.
> Enumerate failure modes before writing any test code. Do not modify production code.
> File ownership: owns `tests/` and any `*.test.*` / `*.spec.*` files.

### Optional roles (spawn only if user requested them)

Copilot CLI: use `agent_type: "general-purpose"` for all optional roles.
OpenCode: use `subagent_type: "general"` for all optional roles.

**Editor** prompt:
> You are the Editor on the surgical team for [PROJECT NAME]. Invoke the `editor` skill
> at the start of this session and before every task you claim.
> Claim tasks tagged `[docs]` from the shared task list in the session SQL database.
> Read the code before writing about it. Never document what you haven't verified.
> File ownership: owns `docs/`, `README.md`, and any `*.md` documentation files.

**Toolsmith** prompt:
> You are the Toolsmith on the surgical team for [PROJECT NAME]. Invoke the `toolsmith`
> skill at the start of this session and before every task you claim.
> Claim tasks tagged `[tooling]` from the shared task list in the session SQL database.
> Every tool you build must have --help text and tests.
> File ownership: owns `scripts/`, `Makefile`, and any tooling/config files.

**Language Lawyer** prompt:
> You are the Language Lawyer on the surgical team for [PROJECT NAME]. Invoke the
> `language-lawyer` skill at the start of this session and before every task you claim.
> Claim tasks tagged `[research]` from the shared task list in the session SQL database.
> Cite the spec or docs — never guess. Say "I don't know" when uncertain.
> File ownership: read-only research; communicate findings, do not edit files.

**Program Clerk** prompt:
> You are the Program Clerk on the surgical team for [PROJECT NAME]. Invoke the
> `program-clerk` skill at the start of this session and before every task you claim.
> Claim tasks tagged `[structure]` from the shared task list in the session SQL database.
> Propose all reorganizations to the Surgeon for approval before executing.
> File ownership: proposes structure changes; executes only after Surgeon approval.

Spawn all agents in a single response using parallel `task` tool calls.

## Step 5: Confirm and Hand Off

After spawning, tell the user:

> Team is live. You are the Surgeon — claim `[implement]` tasks and your teammates
> will handle review, testing, and support in parallel.
>
> A few reminders:
> - Two teammates must not edit the same file — follow the ownership assignments above
> - Run the `assemble-team` skill at any time for a full role reference
> - Use `/tasks` to monitor background teammates
> - When the work is done, cancel any idle background tasks and clean up team resources
