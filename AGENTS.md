# Brooks Surgical Team

This repository implements Fred Brooks' Surgical Team model from *The Mythical Man-Month* as composable agent skills for Claude Code, GitHub Copilot CLI, OpenCode, OpenAI Codex, and Hermes Agent.

## Project Structure

```
.codex/                  — Codex configuration and agent role definitions
  config.toml            — Optional subagent runtime limits
  agents/                — Per-role TOML configs for specialist subagents
.opencode/agents/        — OpenCode agent dispatch files
.github/agents/          — GitHub Copilot CLI custom agent files
agents/                  — Claude Code agent dispatch files
commands/                — Claude Code slash commands
.agents/skills/          — Skill discovery path (symlinks to skills/)
skills/                  — Surgical team skill definitions (SKILL.md per role)
.claude-plugin/          — Claude Code plugin manifest
```

## The Eight Roles

| Role | Responsibility |
|---|---|
| **Surgeon** | Chief programmer — owns all implementation decisions |
| **Copilot** | Code review against design intent — never edits source |
| **Tester** | Adversarial test strategy — assumes the code is wrong |
| **Editor** | Documentation — never documents what hasn't been verified |
| **Toolsmith** | Automation tools — builds what makes the Surgeon faster |
| **Language Lawyer** | Language/framework edge cases — cites spec, never guesses |
| **Program Clerk** | Code organization — proposes before executing |
| **Administrator** | Task tracking and project coordination |

## Working in This Repo

- The Surgeon (main agent) owns all production code changes
- **Dispatchable subagents:** In Codex, the seven specialist roles (Copilot, Tester, Editor, Toolsmith, Language Lawyer, Program Clerk, Administrator) are defined as standalone TOML files in `.codex/agents/` and can be spawned through explicit delegation prompts; `/agent` is used to switch between active agent threads. On Claude Code, Copilot CLI, and OpenCode, only Copilot, Tester, and Language Lawyer have standalone dispatch adapter files. Claude Agent Teams can still spawn additional prompted teammates, but that team surface is distinct from ordinary subagent dispatch; the remaining roles otherwise run inline via their skills on those platforms.
- The Language Lawyer can be invoked in two ways: **inline** via the `language-lawyer` skill for quick lookups, or **dispatched as a subagent** for deeper investigation that should not block the Surgeon. When dispatched, Codex inherits the parent session's sandbox and approval policy; Claude Code, Copilot CLI, and OpenCode grant the role shell/web lookup access without file edit access.
- Skills in `skills/*/SKILL.md` follow the Agent Skills standard and are designed for compatibility with Claude Code, GitHub Copilot CLI, OpenCode, Codex, and Hermes Agent
- Do not modify skill files and platform-specific agent files simultaneously in the same change — they have separate concerns
- Hermes Agent has no on-disk per-role agent-definition format at all (no `.codex/agents/`-style directory applies) — its own specialist dispatch is ephemeral, via `delegate_task` at call time. See `## Multi-Agent (Hermes Agent)` below and `skills/assemble-with-hermes-team/SKILL.md`.

## For Any Coding Agent

- Treat `skills/*/SKILL.md` as the canonical role definitions.
- Treat `.codex/`, `.github/agents/`, `.opencode/agents/`, and `agents/` as platform adapters.
- Keep platform adapters behaviorally aligned with the relevant skill, but avoid coupling skill changes and adapter changes in one commit unless the change explicitly requires both.
- Preserve the Surgeon-led model: the main agent owns implementation decisions; specialists review, test, research, document, organize, or coordinate.
- Verify platform-specific claims against the relevant platform docs or local runtime before updating instructions.

## Multi-Agent (Codex)

Multi-agent support is stable and enabled by default in current Codex CLI releases. Project-scoped custom agents live in `.codex/agents/`; global personal agents live in `~/.codex/agents/`.

See `.codex/agents/` for the Codex specialist role definitions. The Surgeon role has no corresponding agent file — it is the default Codex session, not a dispatchable specialist. `.codex/config.toml` only sets optional runtime limits such as `agents.max_concurrent_threads_per_session`.

## Multi-Agent (Hermes Agent)

[Hermes Agent](https://github.com/NousResearch/hermes-agent) has no persistent per-role agent-definition file — there is no equivalent of `.codex/agents/*.toml`. It has two coordination primitives instead:

- **`delegate_task`** — ephemeral in-process dispatch (`goal`, `context`, `output_schema`, `role: "leaf" | "orchestrator"` passed at call time only, nothing on disk). This is what `skills/assemble-with-hermes-team/SKILL.md` uses to spawn each specialist, pasting the relevant skill's body in as `context` since there's no named agent to reference. `delegation.max_concurrent_children`/`max_iterations`/`max_spawn_depth` cap this, set in the user's own `~/.hermes/config.yaml` (not a repo file).
- **Kanban board** (`kanban_create`/`kanban_list`/`kanban_link`/`kanban_unblock`) — a durable, SQLite-backed task board shared across all Hermes profiles/sessions on the machine, with dependency-aware states (`triage → todo → ready → running → blocked → review → done → archived`). It's the closest Hermes equivalent to Claude Code's `TaskCreate`/`TaskList`/`TaskUpdate`/`TaskGet`, and outlives a single session. **Neither the orchestrator nor the worker set of `kanban_*` tools is available by default**: per Hermes' own docs, "a regular `hermes chat` session has zero `kanban_*` tools in its schema unless the active profile explicitly enables the `kanban` toolset for orchestrator work" — the exact profile-config syntax for that isn't pinned down in Hermes' public docs as of this writing. The `HERMES_KANBAN_TASK`-gated worker tools (`kanban_complete`, `kanban_comment`, `kanban_block`, etc.) are separately injected only into tasks the kanban dispatcher itself spawns (a configured Hermes profile per role). A `delegate_task` child gets neither set automatically. See `skills/assemble-with-hermes-team/SKILL.md` for the full workflow, including the fallback when the current profile has no `kanban` toolset enabled.

Hermes' own project-local skill directories (`.agents/skills/` or `.hermes/skills/`) require a one-time `hermes skills trust <path>` before they load — otherwise skill discovery here needs no adapter at all. For a permanent setup that needs no trust step and works from any project, add this repo's `skills/` path to `skills.external_dirs` in `~/.hermes/config.yaml` instead (supports `~`/`${VAR}` expansion; a project-local skill of the same name still wins over an `external_dirs` one).

## AGENTS.md Discovery

Codex loads `AGENTS.md` files using the following precedence:

1. **Global:** `~/.codex/AGENTS.override.md` suppresses `~/.codex/AGENTS.md`; otherwise `~/.codex/AGENTS.md` is loaded first if present.
2. **Project:** Walking from the git root to the current working directory, at each level Codex checks `AGENTS.override.md`, then `AGENTS.md`. At most one project instruction file is loaded per directory; files closer to cwd appear later and win on conflicts.
3. **Size limit:** Total combined size limit: 32 KiB (`project_doc_max_bytes`) — Codex stops including AGENTS.md files once the concatenated size reaches this cap.

Use `AGENTS.override.md` in a subdirectory to replace this file's guidance with context specific to that part of the codebase.

Hermes Agent also reads `AGENTS.md`, with its own similar-but-distinct precedence: repo-wide `AGENTS.md` loads first, then package-level, then the most-specific `AGENTS.md` in the cwd (which wins); it also discovers parent-directory `AGENTS.md` files lazily as the agent reads into new subdirectories mid-session, rather than loading everything upfront. An `AGENTS.override.md` next to an `AGENTS.md` loads instead of the committed file, same concept as Codex's override. `delegate_task` children inherit the parent session's already-resolved project-context chain.
