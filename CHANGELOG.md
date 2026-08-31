# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- OpenAI Codex compatibility: `.codex/agents/*.toml` custom agents for all seven specialist roles, `.codex/config.toml` subagent runtime settings, `AGENTS.md` for Codex repository guidance, and a `.agents/skills/` symlink mirror for Codex skill discovery
- **Grok Build support**
  - New `assemble-with-grok-team` skill for native parallel surgical team spawning using Grok's `spawn_subagent`, worktree isolation (`isolation: "worktree"` for writers), and shared `todo_write` task tracking (visible via Ctrl+T).
  - Three new Grok-native agent definitions in `.grok/agents/`: `brooks-copilot` (read-only review), `brooks-tester` (adversarial testing in worktree), and `brooks-language-lawyer` (spec-driven research).
  - `.grok/agents/README.md` documenting discovery, manual dispatch, and cross-platform maintenance notes.
  - Updated README with Grok installation guidance, parallel spawn instructions, and repository structure documentation.
  - Minor cross-references added to `assemble-team` and `using-brooks-team` skills; `assemble-with-fleet` recursion guard now also covers `assemble-with-grok-team`.

### Fixed

- Language Lawyer subagent (`agents/language-lawyer.md`, `.github/agents/language-lawyer.agent.md`) had no file read access on Claude Code and Copilot CLI, despite its protocol requiring it to read the code triggering the question
- Copilot subagent had no enforced read-only restriction on Claude Code and Copilot CLI, despite the README documenting it as read-only; only the OpenCode copy actually denied edit access
- `assemble-with-fleet` skill exclusion was missing from the Claude Code copies of the Copilot, Tester, and Language Lawyer dispatch templates (`agents/*.md`), present only in the OpenCode and Copilot CLI copies
- `assemble-team` skill exclusion was missing from all three of the Claude Code, Copilot CLI, and OpenCode copies of the Copilot, Tester, and Language Lawyer dispatch templates; Codex's `[CODEX-STOP]` blocks already excluded it and were used as the reference for backporting this fix
- Language Lawyer permission notes had drifted to three different wordings across `agents/`, `.opencode/agents/`, and `.github/agents/`
- `.codex/config.toml` used `agents.max_threads` (a documented legacy alias) and an undocumented `agents.max_depth` key with no current basis in the Codex config schema; renamed to `agents.max_concurrent_threads_per_session` and removed the unverified depth key
- Codex documentation links updated from `developers.openai.com/codex` to the current `learn.chatgpt.com` location
- `assemble-team` and `assemble-with-fleet` skill exclusions were missing from all three `.grok/agents/brooks-*.md` guard blocks, which only excluded `assemble-with-grok-team`, `using-brooks-team`, and `surgeon`; backported using the other four platforms' guards as reference
- README's maintainer note on agent-file duplication still said "four locations," omitting `.grok/agents/` after Grok Build landed; corrected to five and added Grok's file
- README's "Two ways to start" table omitted Grok Build from the `assemble-team` single-session row, contradicting the "What lives where" table which already listed it there
- `skills/using-brooks-team/SKILL.md`'s Administrator platform note omitted Grok Build from its inline-only platform list
- README's Claude Code `agents/` row in "What lives where" omitted Language Lawyer from its dispatch-template list
- `.grok/agents/README.md` described the Language Lawyer's access as "web search + fetch allowed," but no distinct fetch tool is granted; corrected to name the actual `web_search`/`run_terminal_command` tools
- Grok Build integration re-verified against 1.0.13 (previously checked against 1.0.3); everything held except `capability_mode` (see above)
- The documented "Local clone (development / testing)" install method for Grok Build was unreliable specifically when `brooks-agent-team` is already installed as a Claude Code plugin, live-verified with a real Claude Code install present: `[plugins].paths` silently loses to the existing install (lowest-priority discovery location) even when also enabled, and `.grok/skills/` symlinks produce duplicate entries for every skill that exists in both versions. Documented a project-scoped `.grok/plugins/<name>` symlink instead, confirmed to cleanly override the existing install with no duplication, and added explicit failure-mode warnings to the two previously-documented methods

### Changed

- `skills/program-clerk/SKILL.md` now notes how this role relates to Brooks' original Program Clerk (a technical records archivist), and why it's refocused on code structure now that version control covers the records-keeping job
- README Inspiration section now names conceptual integrity as Brooks' stated rationale for the surgical team structure, and documents why the two secretary roles are omitted
- README's maintainer note on agent-file duplication is now a concrete checklist of what must stay in sync across platforms, updated to reflect Codex as a fourth platform
- README's Agent Skills standard support statement (Inspiration section) now consistently lists Codex alongside the other three platforms instead of describing it as a separate addendum
- **Grok spawning pattern (2026-06, verified against Grok 0.2.43; superseded 2026-08-14)**: custom `.grok/agents/*.md` roles were observed to receive only a single turn on direct `subagent_type` spawn in the interactive TUI, so `assemble-with-grok-team` originally spawned via the built-in `general-purpose` agent with a lightweight first-turn prompt, then used `resume_from` for deep work. Live headless smoke testing on Grok Build 1.0.3 confirmed this limitation is resolved — all three `brooks-*` roles, dispatched directly by `subagent_type`, completed real multi-tool-call tasks (2-4 tool calls each, including a worktree-isolated write/bash task and a web-research task) fully autonomously in the background, with `resume_from` never required. **The skill was rewritten accordingly:** Copilot, Tester, and Language Lawyer are now spawned directly by name with their full task in one call; only Editor/Toolsmith/Program Clerk (no native Grok agent definition) still spawn via `general-purpose`. `resume_from` is kept solely as an optional follow-up mechanism after a subagent completes.
- `assemble-with-grok-team` documents the full operational surface: monitoring (Ctrl+T todos, Ctrl+G tasks, Ctrl+; queue, `get_command_or_subagent_output`), copyable single-call JSON spawn templates for all roles (including optional Editor/Toolsmith/Program Clerk), a complete worktree review/apply workflow (`x.ai/git/worktree/*` extension methods with manual-git fallback), conflict handling, and a Surgeon checklist for serializing writing teammates.
- Copilot/Tester/Language Lawyer spawn prompts point at their `.grok/agents/brooks-*.md` role contract (kept in sync with the cross-platform `copilot`/`tester`/`language-lawyer` skills) rather than instructing the teammate to invoke the skill — the agent definition's body already governs behavior once spawned by `subagent_type`. Editor/Toolsmith/Program Clerk prompts still invoke their skill explicitly, since no agent-definition body exists for those three.
- `.grok/agents/*.md` frontmatter carries `name` + `description` + (for Copilot and Language Lawyer) `tools:`, the confirmed real restriction mechanism on Grok Build.
- **`capability_mode` is not a `spawn_subagent` parameter (corrected 2026-09-01).** Earlier rounds of this integration passed `capability_mode` (`read-only`/`read-write`/`execute`/`all`) on every spawn call and found, via live testing, that its Execute axis was enforced but its Write axis was not. Rechecking against current Grok documentation found the deeper issue: `capability_mode` was never a real `spawn_subagent` argument at all. The current docs list the real parameters as `prompt`, `description`, `subagent_type`, `background`, `isolation`, and `resume_from`. All `capability_mode` spawn-JSON keys and related prose were removed; the confirmed working restriction remains frontmatter `tools:` (real hard write-block for Copilot, partial for Language Lawyer since its required Bash access reopens `search_replace` as a side effect), and no restriction for Tester (intentional, matching every other platform).
- **Added `tools:` frontmatter to close the real gap (2026-08-14), confirmed working via live testing:** `brooks-copilot.md` now restricts to `read_file, list_dir, search_tool, use_tool` — verified both the `write` and `search_replace` tools are genuinely unavailable, giving Copilot the first actually-enforced (not just prompt-level) read-only boundary among the Brooks roles. `brooks-language-lawyer.md` adds `web_search, run_terminal_command` to the same base set for its documented Bash-experiment allowance — but testing found `run_terminal_command`'s presence also unlocks `search_replace` as a side effect, so this role's write boundary remains prompt-contract-only, same as before this fix; closing it fully would require dropping Bash access, a tradeoff not made here.
- `administrator` skill task-tracking wording made platform-neutral (`TodoWrite` on Claude Code, `todo_write` on Grok Build).

## [1.1.1] - 2026-05-15

### Fixed

- `.claude-plugin/plugin.json` version was not updated for the v1.1.0 release

## [1.1.0] - 2026-05-15

### Fixed

- OpenCode install section: corrected install path from `~/.opencode/plugins/` to a clone-anywhere + symlink approach; added `mkdir -p` prerequisites before `ln -sf` and `cp` commands

### Changed

- Copilot CLI install section restructured into "Via marketplace (recommended)" and "Manual install (alternate)" subsections, matching the Claude Code section structure; marketplace install via `zakame/skills-marketplace` is now the primary path with `copilot plugin update` for upgrades
- Copilot CLI custom agents are now auto-discovered after `copilot plugin install`; manual copy to `~/.copilot/agents/` documented as fallback only
- `.claude-plugin/` manifest directory now recognized by both Claude Code and Copilot CLI for marketplace loading
- OpenCode install section restructured into "Skills" and "Subagent roles" subsections; corrected install path from `~/.opencode/plugins/` to a clone-anywhere + symlink approach targeting `.opencode/skills/` or `~/.config/opencode/skills/`
- OpenCode doc links updated to specific pages (`/docs/agents/`, `/docs/skills/`)
- OpenCode compatibility section: permission key descriptions clarified (`edit`, `bash`, `webfetch`, `read` with their exact tool scopes); Language Lawyer permission profile added

## [1.0.0] - 2026-05-13

### Added

**Core surgical team skills (Agent Skills standard — all platforms)**

- `surgeon` skill — Chief programmer operating guide: staying focused, making decisive architectural choices, and knowing when to summon supporting roles
- `copilot` skill — Structured code review as the Surgeon's trusted alter ego, reviewing against design intent rather than code style
- `tester` skill — Adversarial test strategy; assumes the Surgeon's code is wrong until proven otherwise
- `administrator` skill — Task tracking, scope defense, and blocker management so the Surgeon stays on the critical path
- `editor` skill — Documentation accuracy and completeness; owns all prose that describes how the system works
- `toolsmith` skill — Builds custom utilities that make the Surgeon permanently faster; triggers when the same manual operation recurs
- `language-lawyer` skill — Language and framework edge cases where being wrong causes subtle bugs; cites the spec, not the assumption
- `program-clerk` skill — Code organization, naming consistency, import hygiene, and dependency clarity
- `using-brooks-team` skill — Meta-skill for team orientation and role routing at the start of any session

**Session assembly skills**

- `assemble-team` skill — Single-session briefing: surveys the project and presents a tailored overview of which roles apply (Copilot CLI and OpenCode)
- `assemble-with-fleet` skill — Parallel team spawn via Copilot CLI fleet mode or OpenCode task tool; one independent agent per role with a shared task list

**Claude Code integration**

- `/assemble-team` slash command — Team briefing as a Claude Code slash command
- `/assemble-with-agent-teams` slash command — Parallel team spawn via Claude Code Agent Teams (requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`)
- Claude Code dispatch templates for `copilot`, `tester`, and `language-lawyer` subagent roles (`agents/`)
- Plugin manifest (`plugin.json`) for marketplace and `--plugin-dir` loading; installable via `zakame/skills-marketplace`

**GitHub Copilot CLI integration**

- Custom agent definitions for `copilot`, `tester`, and `language-lawyer` roles in `.github/agents/` — copyable to `~/.copilot/agents/` for cross-project use
- `assemble-team` and `assemble-with-fleet` skills adapted for Copilot CLI fleet mode

**OpenCode integration**

- OpenCode agent definitions for `copilot`, `tester`, and `language-lawyer` roles in `.opencode/agents/`; use OpenCode's `mode: subagent` and `permission` frontmatter
- Permissions aligned per role: Copilot denies `edit`/`bash`/`webfetch`; Tester allows `edit`/`bash`; Language Lawyer allows `bash`/`webfetch`, denies `edit`
- `SUBAGENT-STOP` guards in all skills to prevent recursive subagent invocation

**Project foundation**

- MIT License
- `README.md` with full installation, usage, and repository structure documentation for all three platforms

### Fixed

- Surgeon skill `description` frontmatter value quoted to prevent YAML parsing issues

[Unreleased]: https://github.com/zakame/brooks-agent-team/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/zakame/brooks-agent-team/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/zakame/brooks-agent-team/releases/tag/v1.0.0
