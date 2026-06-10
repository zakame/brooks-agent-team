# Brooks Surgical Team

This repository implements Fred Brooks' Surgical Team model from *The Mythical Man-Month* as composable agent skills for Claude Code, GitHub Copilot CLI, OpenCode, and OpenAI Codex.

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
- Skills in `skills/*/SKILL.md` follow the Agent Skills standard and are designed for compatibility with Claude Code, GitHub Copilot CLI, OpenCode, and Codex
- Do not modify skill files and platform-specific agent files simultaneously in the same change — they have separate concerns

## For Any Coding Agent

- Treat `skills/*/SKILL.md` as the canonical role definitions.
- Treat `.codex/`, `.github/agents/`, `.opencode/agents/`, and `agents/` as platform adapters.
- Keep platform adapters behaviorally aligned with the relevant skill, but avoid coupling skill changes and adapter changes in one commit unless the change explicitly requires both.
- Preserve the Surgeon-led model: the main agent owns implementation decisions; specialists review, test, research, document, organize, or coordinate.
- Verify platform-specific claims against the relevant platform docs or local runtime before updating instructions.

## Multi-Agent (Codex)

Multi-agent support is stable and enabled by default in current Codex CLI releases. Project-scoped custom agents live in `.codex/agents/`; global personal agents live in `~/.codex/agents/`.

See `.codex/agents/` for the Codex specialist role definitions. The Surgeon role has no corresponding agent file — it is the default Codex session, not a dispatchable specialist. `.codex/config.toml` only sets optional runtime limits such as `agents.max_concurrent_threads_per_session`.

## AGENTS.md Discovery

Codex loads `AGENTS.md` files using the following precedence:

1. **Global:** `~/.codex/AGENTS.override.md` suppresses `~/.codex/AGENTS.md`; otherwise `~/.codex/AGENTS.md` is loaded first if present.
2. **Project:** Walking from the git root to the current working directory, at each level Codex checks `AGENTS.override.md`, then `AGENTS.md`. At most one project instruction file is loaded per directory; files closer to cwd appear later and win on conflicts.
3. **Size limit:** Total combined size limit: 32 KiB (`project_doc_max_bytes`) — Codex stops including AGENTS.md files once the concatenated size reaches this cap.

Use `AGENTS.override.md` in a subdirectory to replace this file's guidance with context specific to that part of the codebase.
