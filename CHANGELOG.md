# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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

[Unreleased]: https://github.com/zakame/brooks-agent-team/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/zakame/brooks-agent-team/releases/tag/v1.0.0
