# Brooks Surgical Team Agents (Grok Native)

This directory contains custom agent definitions for the Brooks Surgical Team when running under **Grok Build**.

These agents serve two purposes:
- They are the canonical, detailed role contract definitions for the Brooks Surgical Team on Grok Build.
- The `assemble-with-grok-team` skill spawns them directly by name (`subagent_type: "brooks-copilot"` etc.) for the actual parallel spawns — confirmed via live smoke testing that Grok 1.0.3 runs them fully autonomously through multi-tool-call tasks, no `general-purpose` indirection needed.

You can also dispatch them directly by name for manual, one-off use outside the skill.

## Available Agents

| Agent | Name | Role | Capability | Notes |
|-------|------|------|------------|-------|
| `brooks-copilot.md` | `brooks-copilot` | Design-intent code review | Read-only | Never edits files. Strong emphasis on spec alignment and finding what the Surgeon missed. |
| `brooks-tester.md` | `brooks-tester` | Adversarial test strategy & implementation | Execute + test writes | Runs in an isolated worktree. Owns only test files. |
| `brooks-language-lawyer.md` | `brooks-language-lawyer` | Language & framework edge cases | Read-only research | Web search and shell access allowed (`web_search`, `run_terminal_command`). Must cite specs, never guess. |

## Usage

### Recommended: Via the Assembly Skill (Revised Spawning Pattern)

```text
Use the assemble-with-grok-team skill
```

This skill implements the current practical pattern for Grok Build (as of 2026-08-14, verified against Grok 1.0.3):

- Each spawned teammate gets its full task in a single `spawn_subagent` call — no lightweight-first-turn/resume split. Copilot, Tester, and Language Lawyer are spawned directly by `subagent_type` (`brooks-copilot`, `brooks-tester`, `brooks-language-lawyer`); their role contract is the body of the `.grok/agents/brooks-*.md` file itself. Editor/Toolsmith/Program Clerk have no native Grok agent definition, so they spawn via `general-purpose` with an explicit skill-invocation instruction in the prompt instead.
- The Surgeon monitors via Ctrl+T (todos pane), Ctrl+G (tasks pane), and the queue pane (Ctrl+;), and retrieves each teammate's result with `get_command_or_subagent_output`.
- `resume_from` remains available for a genuine follow-up round after a teammate's task completes, but is no longer required to get a subagent's first task done.
- The detailed Brooks Surgical Team contracts, file ownership rules, and protocols live in the bodies of these `brooks-*.md` definitions.

This supersedes the general-purpose + lightweight-first-turn + `resume_from` workaround used while the single-turn limitation on Grok 0.2.43 was unconfirmed for 1.0.3 (see the copyable JSON spawn templates in the assemble skill for the exact current wording).

### Manual / Direct Dispatch (Advanced)

You can also dispatch a named `brooks-*` agent directly via the catalog or `spawn_subagent` with `subagent_type: "brooks-copilot"` etc. — the same direct-dispatch pattern the skill itself uses. This is useful for quick one-shot research or when you want the named role contract applied to a single focused task outside the full team spawn.

Give it the full task in the initial prompt; it runs multi-turn autonomously and reports back. Use `resume_from` with its subagent ID only if you want a genuine follow-up round afterward (see the monitoring section in `skills/assemble-with-grok-team/SKILL.md`). The full role contract in the agent body applies either way.

Example manual dispatch prompt:

```text
Dispatch the brooks-copilot agent to review the new auth changes against the original spec. Here is the diff and context...
```

When dispatching manually (or on resume), still provide:
- Clear context on what was implemented / what the spec was
- The relevant todo items (or let the agent claim from the shared list via `todo_write`)
- Explicit file ownership boundaries

The assemble-with-grok-team skill is the supported entry point for the full parallel Surgical Team experience.

## Installation / Discovery

### Local Development (this repo)

> **Note:** `.grok/agents/` discovery is documented in Grok Build's subagent docs (project- and user-scope `.md` files). Frontmatter schema details (e.g. `tools:`, `mcpInheritance:`) can still change faster than the discovery path itself, so re-verify with `grok inspect` after Grok Build updates.

Project-scope discovery is confirmed working: with your CWD inside this repository, `grok inspect` lists `brooks-copilot`, `brooks-tester`, and `brooks-language-lawyer` as `project` agents (verified on Grok 1.0.3). User-scope discovery from `~/.grok/agents/` follows the same convention — confirm with `grok inspect` after copying agents there.

### As a Plugin

When `brooks-agent-team` is installed as a plugin, Grok should surface project-level agents from the plugin's `.grok/agents/` directory (similar to how skills are discovered).

If the agents do not appear after plugin install:
1. Check with `/agents` or the agent catalog (Ctrl+Shift+A in the TUI).
2. As a fallback, symlink or copy the desired agent definitions into `~/.grok/agents/`.

### User-Level Installation

Copy individual agents you want globally available:

```bash
mkdir -p ~/.grok/agents
cp .grok/agents/brooks-*.md ~/.grok/agents/
```

## Design Notes

- These files are role-contract documentation artifacts for the Brooks Surgical Team on Grok, and — since `assemble-with-grok-team` spawns by `subagent_type` directly — their body text is also the effective system prompt the spawned agent runs with. As of Grok 1.0.3, agent frontmatter supports a `tools:` field (restricts the subagent's available tools) and `mcpInheritance:` (controls which parent MCP servers a spawned subagent inherits) — `color:`/`model:` remain absent from Grok's agent schema (Claude Code-only concepts; `model` on Grok is a skill-frontmatter override, not an agent one). **`tools:` is confirmed real, working enforcement (2026-08-14)** — live testing verified it genuinely removes tools from a subagent's available set (not just `write`, but also the separate `search_replace` edit tool) — with one caveat: including `run_terminal_command` in the allowlist was found to also unlock `search_replace` as a side effect, so a role that needs both shell access and a hard write-block cannot currently have both (see `brooks-language-lawyer.md`'s note).
- **Direct dispatch (confirmed 2026-08-14):** these definitions were originally built around an observed single-turn limit on custom named agents in Grok 0.2.43's interactive TUI subagent spawns, which is why `assemble-with-grok-team` used to prefer the `general-purpose` built-in for initial dispatch. Live headless smoke testing confirmed Grok 1.0.3 no longer has this limitation: all three `brooks-*` roles, dispatched directly by `subagent_type`, completed real multi-tool-call tasks (2-4 tool calls each) fully autonomously in the background without ever needing `resume_from`. `assemble-with-grok-team` now spawns these three roles directly by name.
- The Copilot and Language Lawyer are intended as read-only; the Tester is write-capable within test directories and run in a worktree. `isolation` (worktree) is enforced by the platform. **`capability_mode` is not a `spawn_subagent` parameter (corrected 2026-09-01)**. An earlier round of this integration treated it as one and found its Write axis unenforced; current Grok documentation confirms it was never a spawn-time argument at all. The real hard-enforcement mechanism is frontmatter `tools:`, confirmed working for `brooks-copilot` (full write-block) and partial for `brooks-language-lawyer` (Bash access reopens `search_replace` as a side effect).
- **Frontmatter `tools:` is confirmed real, working enforcement (2026-08-14) — with one caveat.** `brooks-copilot.md` now carries `tools: read_file, list_dir, search_tool, use_tool` (no write/edit/shell); live testing confirmed both the `write` and `search_replace` tools are genuinely unavailable to it and every write attempt failed — a real hard read-only boundary, the first on this platform. `brooks-language-lawyer.md` carries the same set plus `web_search` and `run_terminal_command` (needed for its documented Bash-experiment allowance and cross-platform parity) — but including `run_terminal_command` was found to also unlock `search_replace` as a side effect, so **this role's write boundary is not fully closed**; it still relies on its own prompt contract, same as before this fix. Removing `run_terminal_command` would close the gap but sacrifice the role's Bash allowance — a real tradeoff, not yet made.
- All three agents understand the Brooks task tagging convention (`[review]`, `[test]`, `[research]`, etc.) and are expected to participate in the shared `todo_write` list (the single source of truth, visible via Ctrl+T).
- The `<SUBAGENT-STOP>` guard prevents accidental recursion (an agent trying to become the Surgeon or spawn the whole team).
- For maximum parallelism and depth on Grok today: spawn with the assemble skill (direct `subagent_type` dispatch with the full task, no resume needed for the first result) and monitor via `get_command_or_subagent_output`. The named brooks-* definitions remain valuable for catalog discoverability (Ctrl+Shift+A), manual one-offs, and maintaining the role contracts in one place.

## Relationship to Other Agent Definitions

This project also ships agent definitions for other platforms:

- `agents/` — Claude Code dispatch templates
- `.github/agents/` — GitHub Copilot CLI custom agents
- `.opencode/agents/` — OpenCode agents

The Grok definitions in `.grok/agents/` are canonical for the Grok environment only. The shared role protocols originate in the cross-platform skills (`skills/copilot/`, `skills/tester/`, `skills/language-lawyer/`); when review, test, or research protocols change there, keep these files (and the other platform ports) in sync.

## Contributing

When modifying these agents:

1. Keep the surgical team philosophy: the agent serves the Surgeon's focus and never makes architectural decisions.
2. Preserve the strict file ownership and capability boundaries.
3. Update the corresponding skill prompts in `skills/assemble-with-grok-team/` if the contract changes.
4. If behavior changes meaningfully, consider updating the other platform definitions for consistency.