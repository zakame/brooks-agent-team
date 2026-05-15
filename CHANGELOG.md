# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
