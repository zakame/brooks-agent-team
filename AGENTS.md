# Brooks Surgical Team

This repository implements Fred Brooks' Surgical Team model from *The Mythical Man-Month* as composable agent skills for Claude Code, GitHub Copilot CLI, OpenCode, OpenAI Codex, Hermes Agent, and Pi Coding Agent.

## Project Structure

```
.codex/                  — Codex configuration and agent role definitions
  config.toml            — Optional subagent runtime limits
  agents/                — Per-role TOML configs for specialist subagents
.opencode/agents/        — OpenCode agent dispatch files
.github/agents/          — GitHub Copilot CLI custom agent files
.pi/agents/              — Pi Coding Agent subagent files (only take effect with the optional `subagent` example extension)
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
- Skills in `skills/*/SKILL.md` follow the Agent Skills standard and are designed for compatibility with Claude Code, GitHub Copilot CLI, OpenCode, Codex, Hermes Agent, and Pi Coding Agent
- Do not modify skill files and platform-specific agent files simultaneously in the same change — they have separate concerns
- Hermes Agent has no on-disk per-role agent-definition format at all (no `.codex/agents/`-style directory applies) — its own specialist dispatch is ephemeral, via `delegate_task` at call time. See `## Multi-Agent (Hermes Agent)` below and `skills/assemble-with-hermes-team/SKILL.md`.
- Pi Coding Agent core has no on-disk per-role agent-definition format and no subagent dispatch tool either — `.pi/agents/*.md` only takes effect if a user has installed the optional `subagent` example extension. See `## Multi-Agent (Pi Coding Agent)` below and `skills/assemble-with-pi-team/SKILL.md`.

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

- **`delegate_task`** — ephemeral in-process dispatch (`goal`, `context`, `output_schema` passed at call time only, nothing on disk; a `role: "leaf" | "orchestrator"` field is documented but not confirmed present on every deployed version's live tool schema, so don't depend on it). This is the default mode `skills/assemble-with-hermes-team/SKILL.md` uses to spawn each specialist, pasting the relevant skill's body in as `context` since there's no named agent to reference. `delegation.max_concurrent_children`/`max_iterations`/`max_spawn_depth` cap this, set in the user's own `~/.hermes/config.yaml` (not a repo file).
- **Kanban dispatcher** — a durable, SQLite-backed task board shared across all Hermes profiles/sessions on the machine, with dependency-aware states (`triage → todo → ready → running → blocked → review → done → archived`). It's the closest Hermes equivalent to Claude Code's `TaskCreate`/`TaskList`/`TaskUpdate`/`TaskGet`, and outlives a single session. The orchestrator-side `kanban_*` tools (`kanban_create`/`kanban_list`/`kanban_link`/`kanban_unblock`) aren't available by default: per Hermes' own docs, "a regular `hermes chat` session has zero `kanban_*` tools in its schema unless the active profile explicitly enables the `kanban` toolset for orchestrator work." Even without that toolset, the `hermes kanban` shell CLI can drive the same board directly — `skills/assemble-with-hermes-team/SKILL.md` treats this CLI-driven mode as a first-class second path alongside `delegate_task`, not just a fallback. The `HERMES_KANBAN_TASK`-gated worker tools (`kanban_complete`, `kanban_comment`, `kanban_block`, etc.) go to any task the dispatcher itself launches — a shared `default` profile included, not only a dedicated per-role one; a dedicated profile just gives a role a persistent identity instead of sharing `default`. A `delegate_task` child gets neither the orchestrator nor the worker set automatically. See `skills/assemble-with-hermes-team/SKILL.md` for the full workflow, including the `todo_list`/plain-notes fallback when neither kanban path applies.

Hermes' own project-local skill directories (`.agents/skills/` or `.hermes/skills/`) require a one-time `hermes skills trust <path>` before they load — otherwise skill discovery here needs no adapter at all. For a permanent setup that needs no trust step and works from any project, add this repo's `skills/` path to `skills.external_dirs` in `~/.hermes/config.yaml` instead (supports `~`/`${VAR}` expansion; a project-local skill of the same name still wins over an `external_dirs` one).

## Multi-Agent (Pi Coding Agent)

[Pi Coding Agent](https://github.com/earendil-works/pi/tree/main/packages/coding-agent) (`@earendil-works/pi-coding-agent`, verified against v1.0.2) supports the Agent Skills standard natively, so skill discovery needs **zero adapter files**. A project-local `.agents/skills/` is trust-gated, like `.pi/settings.json`, `.pi/mcp.json`, and `.pi/skills/` (`docs/security.md`, "Resources protected by project trust"). Three ways to avoid the trust prompt: symlink `skills/` into the global `~/.agents/skills/` (never trust-gated), pass `--skill <path>` for one session, or pass `--approve` to trust the project for one process. See the [README's Pi Coding Agent section](README.md#pi-coding-agent). Pi reads this file through its `AGENTS.md`/`CLAUDE.md` context-file mechanism, which ignores trust.

Pi core has no subagent tool and no shared task list. Built-in `codemode` runs tool calls in parallel inside one conversation; it does not spawn agents. The only multi-agent path is the opt-in `examples/extensions/subagent/` extension, which ships in the npm package but is not installed by default (unchanged from v0.87.1 to v1.0.2). It spawns one OS process per task (`pi --mode json -p --no-session`). The mode depends on which parameter you pass: `agent`+`task` runs one task, `tasks[]` runs up to 8 tasks with 4 at a time (hardcoded), and `chain[]` runs tasks in sequence. There is no `mode` field. Agent files are Markdown with required `name`/`description` and optional `tools`/`model` frontmatter, found in `~/.pi/agent/agents/` (user scope, the default) or in `.pi/agents/` when the call passes `agentScope: "project"`/`"both"`. The file body is the only way to set a role's prompt: it is passed via `--append-system-prompt`, and no call parameter can replace it. This repo ships `.pi/agents/copilot.md`, `tester.md`, and `language-lawyer.md`, which do nothing without the extension. Children don't inherit `--approve` or `--skill`. Unless the project's trust is saved (`/trust`) or `defaultProjectTrust` is `"always"`, they load only global `~/.agents/skills/`. Install the skills there, or put everything the role needs in its `task`. The source repo also has an experimental durable-harness `subagent` tool (`src/experimental/durable/`) that takes only `task`. The npm package excludes it, and it can't load `.pi/agents/`. See `skills/assemble-with-pi-team/SKILL.md`.

Pi has no enforced read-only sandbox. `docs/security.md` says safety comes from OS-level isolation. `tools:` in an agent file is a real allow-list, so Copilot's and Language Lawyer's agent files leave out `edit`/`write`. Before running project-local agents in an untrusted project, the extension asks for interactive confirmation. The prompt is skipped without a UI or with `confirmProjectAgents: false`. This check lives in the extension; it is not a core trust gate.

## AGENTS.md Discovery

Codex loads `AGENTS.md` files using the following precedence:

1. **Global:** `~/.codex/AGENTS.override.md` suppresses `~/.codex/AGENTS.md`; otherwise `~/.codex/AGENTS.md` is loaded first if present.
2. **Project:** Walking from the git root to the current working directory, at each level Codex checks `AGENTS.override.md`, then `AGENTS.md`. At most one project instruction file is loaded per directory; files closer to cwd appear later and win on conflicts.
3. **Size limit:** Total combined size limit: 32 KiB (`project_doc_max_bytes`) — Codex stops including AGENTS.md files once the concatenated size reaches this cap.

Use `AGENTS.override.md` in a subdirectory to replace this file's guidance with context specific to that part of the codebase.

Hermes Agent also reads `AGENTS.md`, with its own similar-but-distinct precedence: repo-wide `AGENTS.md` loads first, then package-level, then the most-specific `AGENTS.md` in the cwd (which wins); it also discovers parent-directory `AGENTS.md` files lazily as the agent reads into new subdirectories mid-session, rather than loading everything upfront. An `AGENTS.override.md` next to an `AGENTS.md` loads instead of the committed file, same concept as Codex's override. `delegate_task` children inherit the parent session's already-resolved project-context chain.

Pi Coding Agent calls these "context files" and accepts `AGENTS.override.md`, `AGENTS.md`, `AGENTS.MD`, `CLAUDE.md`, or `CLAUDE.MD` — loaded from its agent directory, the working directory, and parent directories of the working directory, applying to that directory and everything below it; `AGENTS.override.md` only replaces a same-directory `AGENTS.md`/`CLAUDE.md`, not an ancestor's. Unlike Codex, no documented size cap exists for these files.
