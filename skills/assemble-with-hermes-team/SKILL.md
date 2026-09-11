---
name: assemble-with-hermes-team
description: "Hermes Agent — Spawn a Brooks Surgical Team using Hermes's delegate_task tool for concurrent role dispatch, tracked on Hermes's native kanban board (kanban_create/kanban_list/kanban_link) when your profile has the kanban toolset enabled, or a plain task list otherwise. Use when running under the Hermes Agent harness and you want true parallel teammates."
---

# Assemble Hermes Surgical Team (Parallel)

<SUBAGENT-STOP>
If you are already a `delegate_task` child, or a kanban-dispatched worker (`HERMES_KANBAN_TASK` set in your environment), you are already playing one of these roles. Do not invoke `assemble-with-hermes-team`, `assemble-with-fleet`, `assemble-with-grok-team`, `using-brooks-team`, or `surgeon`. This skill spawns every role with `role: "leaf"`, and leaf children cannot call `delegate_task` at all (Hermes refuses it outright, independent of any depth setting) — so recursive spawning isn't just discouraged here, it's not physically possible for a `leaf` child. An `orchestrator`-role child could in principle re-delegate, but only up to `delegation.max_spawn_depth` (default `1`, i.e. flat) — don't rely on that either.
</SUBAGENT-STOP>

When invoked, execute the following steps in order.

## Step 0: Prerequisites

- **Skill discovery.** This repo's skills live in `skills/` (mirrored at `.agents/skills/`), a project-local directory Hermes recognizes per the Agent Skills standard. Project-local skill directories require a one-time trust step before Hermes loads them:
  ```
  hermes skills trust <path-to-this-repo>
  ```
  Run once per clone; skip if already trusted. For a permanent setup that needs no trust step and works from any project, add this repo's `skills/` path to `skills.external_dirs` in `~/.hermes/config.yaml` instead (supports `~`/`${VAR}` expansion) — see the [README's Hermes Agent section](../../README.md#hermes-agent).
- **Kanban toolset (check before relying on Step 3).** A regular `hermes chat` session has **zero** `kanban_*` tools by default — per Hermes' own docs, they only appear if "the active profile explicitly enables the `kanban` toolset for orchestrator work." Check now whether `kanban_create`/`kanban_list`/`kanban_link`/`kanban_unblock` are actually in your tool list. If they're not, your profile doesn't have the toolset enabled (the exact config syntax for enabling it isn't pinned down in Hermes' public docs as of this writing — check `hermes profile show <name>` or your Hermes version's current profile documentation), and Step 3 falls back to a plain task list instead of the kanban board.
- **Kanban dispatcher.** Separately from the toolset above, cards only advance on their own (dispatcher-driven profile workers, see "Advanced" below) if the dispatcher is running — on by default via `kanban.dispatch_in_gateway: true` in `~/.hermes/config.yaml` (siblings: `dispatch_interval_seconds: 60`, `review_dispatch: true`). Irrelevant if you don't have the kanban toolset enabled.

## Step 1: Quick Project Survey

Read just enough to understand the work ahead:

- Project name / primary language / framework (README, package.json, Cargo.toml, pyproject.toml, etc.)
- Recent activity: `git log --oneline -5` and current branch
- Presence of `tests/`, `docs/`, `AGENTS.md`, or similar
- Any obvious large modules or risky areas from the top-level structure

Do **not** do a full orientation — this skill is about getting the team spawned quickly. Record a one-paragraph summary for use in `delegate_task` calls.

## Step 2: Decide the Team Composition

Ask the user one focused question:

> I'll spawn the core surgical team now (Surgeon = you + **Copilot** + **Tester**).
>
> Do you want any additional teammates?
>
> - **Language Lawyer** — for tricky language, framework, or version edge cases
> - **Editor** — for documentation-heavy work
> - **Toolsmith** — if you need new automation or scripts
> - **Program Clerk** — if the codebase needs reorganization alongside the work
>
> Reply with the roles to add, or say **"default"** (or just "go") to proceed with Copilot + Tester only.

Wait for the user's explicit response before continuing.

Core team (always spawned): Copilot (read-only review), Tester (adversarial tests).

## Step 3: Initialize the Shared Task Board

Hermes has no in-memory shared-list tool like Claude Code's `TaskCreate`/`TaskList`/`TaskUpdate`/`TaskGet` or Grok's `todo_write`. Its native equivalent is the **kanban board** — a SQLite-backed table shared across every Hermes profile/session on the machine, not scoped to this one conversation. It's real, but it is not zero-setup: per Step 0, use it only if `kanban_create`/`kanban_list`/`kanban_link`/`kanban_unblock` actually appear in your tool list.

**If the kanban toolset is enabled:**

- `kanban_create(title, assignee)` — one card per unit of work. Set `assignee` to the role name (`copilot`, `tester`, `language-lawyer`, `editor`, ...).
- `kanban_link(parent_id, child_id)` — add a dependency edge after creation, e.g. blocking a `[review]` card on its `[implement]` card.
- `kanban_list(...)` — check board state (supports filters, exact set undocumented — inspect what your Hermes version accepts).
- `kanban_unblock(task_id)` — release a card once its blocker resolves.

Card states flow `triage → todo → ready → running → blocked → review → done → archived`; a `kanban_link` edge auto-promotes a card from `todo` to `ready` once every parent card reaches `done`.

Use the same tags the other platform skills use, so cards read consistently across the project:
- `[implement]` — Surgeon (you)
- `[review]` — Copilot
- `[test]` — Tester
- `[research]` — Language Lawyer (if spawned)
- `[docs]` — Editor (if spawned)
- `[tooling]` — Toolsmith (if spawned)
- `[structure]` — Program Clerk (if spawned)

Create at least one `[implement]` card, then its dependent `[review]` and `[test]` cards, then link each to its parent with `kanban_link`, before spawning anyone.

**If the kanban toolset is not enabled:** there's no fallback task-list tool either (same situation as Codex or OpenCode). Just enumerate the plan with the same tags, in your own working notes, and pass the relevant items directly into each `delegate_task` call's `context` in Step 4 — the plan lives in what you tell each child, not in a shared tool.

## Step 4: Spawn the Team (Parallel)

Hermes has no persistent per-role agent-definition file the way Codex's `.codex/agents/*.toml`, OpenCode's `.opencode/agents/*.md`, or Grok's `.grok/agents/*.md` do. `delegate_task` is fully ephemeral: `goal`, `context`, `output_schema`, and `role` (`"leaf"` or `"orchestrator"`) are passed at call time only, with no on-disk registry to point at. Hermes "profiles" do provide named, persistent identities, but each is a full separate Hermes home directory (own config, skills, memory, state db) — too heavy to stand in for a lightweight role file here.

Spawn each role with `delegate_task`, `role: "leaf"`, and the target skill's full body pasted into `context` — since there's no named agent to reference, the skill content itself is the role contract:

```
delegate_task(
  goal: "Review the [FEATURE] implementation against its design intent; report Blocking/Important/Suggestions findings.",
  context: "<paste skills/copilot/SKILL.md body>\n\nProject: [one-paragraph summary]. Diff/files to review: [...]. Do not edit source.",
  role: "leaf"
)
```

Repeat per selected role: `tester` → `skills/tester/SKILL.md`, `language-lawyer` → `skills/language-lawyer/SKILL.md`, `editor` → `skills/editor/SKILL.md`, `toolsmith` → `skills/toolsmith/SKILL.md`, `program-clerk` → `skills/program-clerk/SKILL.md`. Give each child its full task up front (project summary, the card it owns, file ownership, the concrete diff/spec/question) — there is no lightweight-first-turn/resume split to rely on.

`delegation.max_concurrent_children` (default 10, set in the user's `~/.hermes/config.yaml`, not a repo file) governs how many of these run at once. Issue all selected roles' `delegate_task` calls together rather than waiting on each in turn.

**Reconciling with the kanban board (if you have it):** `delegate_task` children are not spawned by the kanban dispatcher, so even if your own profile has the orchestrator-side `kanban` toolset enabled, they don't inherit it — and they don't get the separate `HERMES_KANBAN_TASK`-gated worker tools either (`kanban_show`, `kanban_complete`, `kanban_block`, `kanban_request_review`, `kanban_comment`, etc. — those exist only for a task the dispatcher itself launched). So even with the toolset enabled, nothing in this default flow can mark a card `done`, `blocked`, or commented-on directly.

Treat the board as **plan-of-record, not live status**: create the cards up front for visibility and dependency tracking, `kanban_list` to check them, and `kanban_unblock` a card once its `delegate_task` parent's result confirms it's actually done. The real "is this finished" signal comes from the `delegate_task` result itself, the same as on any other platform without a full task-update tool (Codex, OpenCode). Getting cards to close out through the kanban tools themselves (`kanban_complete`, `kanban_comment`, etc.) needs the dispatcher-driven, profile-based setup described at the end of this skill.

### File Ownership Rules (Include in Every `delegate_task` Context)

- **Copilot**: read-only across the repo. Never edits source.
- **Tester**: owns `tests/`, `**/*.test.*`, `**/*.spec.*`, and test utilities.
- **Language Lawyer**: read-only research. May search the web. Never edits files.
- **Editor**: owns `docs/`, `README*`, `*.md` (except code-adjacent notes).
- **Toolsmith**: owns `scripts/`, tooling configs, Makefiles, new automation. Every tool needs `--help` and tests.
- **Program Clerk**: proposes structure changes only; executes renames/refactors only after Surgeon approval.

No worktree isolation is documented for `delegate_task` children — these rules are enforced by convention and Surgeon review, the same as on OpenCode and Copilot CLI, not by a platform-level sandbox. Never run two roles with overlapping ownership at once.

## Step 5: Handoff

After spawning, tell the user:

> Team is live. You are the Surgeon: continue the critical path while your teammates run as `delegate_task` children.
>
> [If the kanban toolset is enabled:] The plan is on the kanban board (`kanban_list()` any time for state — durable, survives this session ending), but it's plan-of-record only — neither your teammates nor this session can mark a card `done` or comment on it, since those tools are dispatcher-gated. Treat each `delegate_task` result as the real completion signal, and use `kanban_unblock` to release anything it was blocking.
>
> [If not:] There's no shared task-list tool here — the plan lives in what you told each teammate. Track completion from each `delegate_task` result as it returns.
>
> No worktree isolation here — respect the file-ownership rules above; never let two roles touch the same files.
>
> Run `assemble-team` any time for the full role reference.

## Advanced: fully autonomous kanban workers (optional)

If dedicated Hermes profiles already exist for these roles (`~/.hermes/profiles/<role>/`), the kanban dispatcher can spawn and run those profiles against `kanban_create`d cards directly — those workers get the full `kanban_show`/`kanban_complete`/`kanban_block`/`kanban_request_review` lifecycle automatically, closing the loop without the Surgeon updating cards by hand. Setting this up means a full separate Hermes home directory per role, which is out of scope for a quick team assembly; consider it only if Hermes teams are run often enough to justify it.

## Anti-Patterns to Prevent

- Do not spawn the full team for tiny changes.
- Do not let two roles write the same files.
- Do not tell the user a card (or its underlying work) is done without checking the actual `delegate_task` result — the board itself won't reflect it.
- Do not assume a `delegate_task` child, or the Surgeon session, can mark a kanban card `done`/`blocked` or comment on it — those tools are dispatcher-gated; only a dedicated profile-based worker gets them.
