# brooks-agent-team

A skills plugin that organizes AI-assisted software development around Fred Brooks' **Surgical Team** model from *The Mythical Man-Month* (1975). Compatible with **GitHub Copilot CLI**, **Claude Code**, and **OpenCode**.

Instead of every team member working on all parts of a system, the Surgical Team concentrates critical work in one skilled "surgeon" (chief programmer), supported by specialized roles that keep the surgeon focused and productive. This plugin maps those roles to agent skills and dispatch templates.

## Inspiration

This project draws from two sources:

**Fred Brooks' Surgical Team** (Chapter 3, *The Mythical Man-Month*, 1975), originally conceived by Harlan Mills: a small, highly specialized team organized around a single chief programmer who writes all critical code, supported by a copilot, tester, toolsmith, editor, administrator, language lawyer, and program clerk — each with a distinct, non-overlapping responsibility.

**[Superpowers by Jesse Vincent](https://github.com/obra/superpowers)**: a Claude Code skills framework that demonstrated how composable, role-aware skills can guide an AI agent through disciplined software development workflows. The structure, conventions, and plugin format of this project follow Superpowers' design closely.

The `SKILL.md` format used here conforms to the [Agent Skills open standard](https://github.com/agentskills/agentskills), which is supported by GitHub Copilot CLI, Claude Code, and OpenCode.

## The Team

| Role | Skill | Responsibility |
|------|-------|----------------|
| **Surgeon** | `surgeon` | Chief programmer — owns all critical decisions and implementation |
| **Copilot** | `copilot` | Alter ego — reviews all of the Surgeon's work against design intent |
| **Tester** | `tester` | Adversarial quality — finds failure modes the Surgeon was too optimistic to anticipate |
| **Administrator** | `administrator` | Project tracking — manages tasks, blockers, and scope so the Surgeon doesn't have to |
| **Editor** | `editor` | Documentation — ensures everything written about the code is accurate and complete |
| **Toolsmith** | `toolsmith` | Automation — builds utilities that make the Surgeon permanently faster |
| **Language Lawyer** | `language-lawyer` | Edge cases — answers the language/framework questions where being wrong causes subtle bugs |
| **Program Clerk** | `program-clerk` | Code organization — keeps the codebase navigable, names accurate, dependencies clean |

**You are always the Surgeon.** The other roles are invoked as needed — either as inline guidance (the skill guides Claude to perform that role temporarily) or as dispatched subagents (for independent, parallelizable work like code review or test writing).

## Installation

### GitHub Copilot CLI

Clone this repository to a stable location:

```bash
git clone https://github.com/zakame/brooks-agent-team ~/.copilot/plugins/brooks-agent-team
```

Register the skills directory in a Copilot CLI session:

```
/skills add ~/.copilot/plugins/brooks-agent-team/skills
```

This makes all team skills available. To persist across sessions, add the directory permanently using `/skills add`. Skills are immediately usable — Copilot will invoke them automatically based on context, or you can name them explicitly in your prompt:

```
Use the surgeon skill to start implementing this feature.
Use the copilot skill to review my changes.
```

The `copilot` and `tester` **custom agents** are available in `.github/agents/` and can be copied to `~/.copilot/agents/` for use across all projects:

```bash
cp ~/.copilot/plugins/brooks-agent-team/.github/agents/*.agent.md ~/.copilot/agents/
```

Then invoke them via `/agent` or directly in a prompt:

```
Use the copilot agent to review the authentication changes.
```

### OpenCode

Clone this repository to a stable location:

```bash
git clone https://github.com/zakame/brooks-agent-team ~/.opencode/plugins/brooks-agent-team
```

OpenCode auto-discovers agent definitions from `.opencode/agents/` in your project directory. To make the Copilot and Tester subagents available in a project:

```bash
cp ~/.opencode/plugins/brooks-agent-team/.opencode/agents/*.md .opencode/agents/
```

For cross-project use, consult the [OpenCode agent documentation](https://opencode.ai/docs) for global agent configuration paths.

Skills are available via the [Agent Skills open standard](https://github.com/agentskills/agentskills). OpenCode auto-discovers `SKILL.md` files from registered plugin directories — consult the [OpenCode documentation](https://opencode.ai/docs) for how to configure a skills directory. Once available, skills can be invoked directly in a session:

```
Use the surgeon skill to start implementing this feature.
Use the copilot skill to review my changes.
```

### Claude Code

#### For a single session

Start Claude Code with the `--plugin-dir` flag pointing to this repository:

```bash
claude --plugin-dir /path/to/brooks-agent-team
```

Skills are immediately available. To reload after making changes without restarting:

```
/reload-plugins
```

#### Persistent use

Clone this repository to a stable location and add the `--plugin-dir` flag to your shell alias or Claude Code configuration:

```bash
git clone https://github.com/zakame/brooks-agent-team ~/.claude/plugins/brooks-agent-team
```

Then start Claude Code with:

```bash
claude --plugin-dir ~/.claude/plugins/brooks-agent-team
```

## Usage

### Two ways to start

| Skill / Command | Tool | When to use |
|---------|------|-------------|
| `assemble-team` skill | Copilot CLI, Claude Code & OpenCode | Single-session work — one AI instance plays all roles sequentially |
| `/assemble-team` command | Claude Code only | Same as above, as a slash command |
| `assemble-with-fleet` skill | Copilot CLI & OpenCode | Parallel work — spawns one independent session per role (Copilot CLI uses `/fleet`; OpenCode uses the task tool) |
| `/assemble-with-agent-teams` command | Claude Code only | Parallel work — spawns via Claude Code Agent Teams |

### `assemble-team` — single-session briefing

Run at the start of any development session to get a project-contextual briefing on which roles apply and how to invoke them.

**Copilot CLI:**
```
Use the assemble-team skill
```

**Claude Code:**
```
/assemble-team
```

**OpenCode:**
```
Use the assemble-team skill
```

The AI surveys your project and presents a tailored overview of the team. Roles are invoked on demand as the work requires them. Lightweight and works without any additional setup.

### Parallel team spawn

Spawn one independent AI session per role so that Copilot reviews and Tester writes tests while you continue on the critical path.

**Copilot CLI** — uses `assemble-with-fleet` skill (requires experimental fleet mode):
```
Use the assemble-with-fleet skill
```

**OpenCode** — uses `assemble-with-fleet` skill (spawns subagents via the task tool):
```
Use the assemble-with-fleet skill
```

**Claude Code** — uses `/assemble-with-agent-teams` (requires [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams.md)):

Enable Agent Teams first:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```
Then run:
```
/assemble-with-agent-teams
```

**What you get:**
- One independent session per role (Copilot, Tester, and any others you choose)
- True parallelism — Copilot reviews while you continue coding
- Shared task list with dependency tracking and role-tagged tasks (`[implement]`, `[review]`, `[test]`, etc.)
- File ownership assignments to prevent conflicts

### Invoke skills directly

**Copilot CLI** — use the skill name in your prompt:
```
Use the surgeon skill to start implementing this feature.
Use the copilot skill to review my changes.
Use the language-lawyer skill for this edge case.
```

**Claude Code** — skills are namespaced under `brooks-agent-team:`:
```
brooks-agent-team:surgeon
brooks-agent-team:copilot
brooks-agent-team:tester
brooks-agent-team:administrator
brooks-agent-team:editor
brooks-agent-team:toolsmith
brooks-agent-team:language-lawyer
brooks-agent-team:program-clerk
```

**OpenCode** — use the skill name directly in a session prompt:
```
Use the surgeon skill to start implementing this feature.
Use the copilot skill to review my changes.
Use the language-lawyer skill for this edge case.
```

### Automatic invocation

The AI reads each skill's `description` field and automatically invokes the relevant skill when your request matches its trigger conditions. For example:

- Starting an implementation task → `surgeon` skill loads
- Completing a feature → `copilot` skill is suggested
- Spotting repeated manual operations → `toolsmith` skill activates
- Encountering a language edge case → `language-lawyer` skill is invoked

### Dispatch subagent roles

The **Copilot** and **Tester** roles can be dispatched as independent subagents, allowing the Surgeon to continue working on the critical path while review or test writing happens in parallel.

**Copilot CLI** — use the custom agents in `.github/agents/` (copy to `~/.copilot/agents/` for cross-project use):
```
Use the copilot agent to review the authentication changes.
Use the tester agent to write tests for the payment module.
```

**Claude Code** — use the dispatch templates in `agents/copilot.md` and `agents/tester.md`.

**OpenCode** — use the agent definitions in `.opencode/agents/`:
```
Dispatch the Copilot agent to review the authentication changes.
Dispatch the Tester agent to write tests for the payment module.
```

The Copilot agent is read-only (permissions deny `edit`, `bash`, and `webfetch`); the Tester agent can write files and run shell commands (permissions allow `edit` and `bash`).

## Repository Structure

```
skills/                         Shared across all platforms (Agent Skills standard)
  using-brooks-team/SKILL.md      Meta-skill: team orientation and role routing
  surgeon/SKILL.md                Chief programmer operating guide
  copilot/SKILL.md                Code review and backup
  tester/SKILL.md                 Adversarial test strategy
  administrator/SKILL.md          Task tracking and scope defense
  editor/SKILL.md                 Documentation and accuracy
  toolsmith/SKILL.md              Custom tool builder
  language-lawyer/SKILL.md        Language and framework edge cases
  program-clerk/SKILL.md          Code organization and structure
  assemble-team/SKILL.md          Team briefing skill (Copilot CLI and OpenCode)
  assemble-with-fleet/SKILL.md    Parallel team spawn via Copilot CLI fleet mode or OpenCode task tool

.claude-plugin/                 Claude Code specific
  plugin.json                     Plugin manifest (name, version, author)
agents/                         Claude Code dispatch templates
  copilot.md                      Subagent template for Copilot role
  tester.md                       Subagent template for Tester role
commands/                       Claude Code slash commands
  assemble-team.md                /assemble-team command
  assemble-with-agent-teams.md    /assemble-with-agent-teams command

.github/agents/                 GitHub Copilot CLI custom agents
  copilot.agent.md                Code reviewer agent
  tester.agent.md                 Adversarial tester agent

.opencode/agents/               OpenCode custom agents
  copilot.md                      Code reviewer agent
  tester.md                       Adversarial tester agent
```

### What lives where

The `skills/` directory is the core of the plugin. Each subdirectory contains a `SKILL.md` file conforming to the [Agent Skills open standard](https://github.com/agentskills/agentskills), which is supported by GitHub Copilot CLI, Claude Code, and OpenCode. These files work on any platform that reads the standard.

Platform-specific files provide deeper integration:

| Directory | Platform | Purpose |
|-----------|----------|---------|
| `.claude-plugin/` | Claude Code | Plugin manifest for `--plugin-dir` loading |
| `agents/` | Claude Code | Dispatch templates for subagent roles (Copilot, Tester) |
| `commands/` | Claude Code | Slash commands (`/assemble-team`, `/assemble-with-agent-teams`) |
| `.github/agents/` | Copilot CLI | Custom agent definitions for `/agent` dispatch |
| `.opencode/agents/` | OpenCode | Custom agent definitions for subagent dispatch |
| `skills/assemble-team/` | Copilot CLI & OpenCode | Team briefing skill |
| `skills/assemble-with-fleet/` | Copilot CLI & OpenCode | Parallel spawn via fleet mode or task tool |

### OpenCode compatibility

The eight role skills in `skills/` are compatible with [OpenCode](https://opencode.ai) via the [Agent Skills open standard](https://github.com/agentskills/agentskills) — OpenCode reads `SKILL.md` files the same way Claude Code and Copilot CLI do.

Full agent dispatch is supported through `.opencode/agents/`, which includes Copilot and Tester subagent definitions. These files use OpenCode's agent frontmatter format (`description`, `mode: subagent`, `permission` block). The permissions match each role's responsibilities: the Copilot denies edits and shell/web fetches, while the Tester allows edits and shell commands. File reads are allowed by default in OpenCode — set `permission.read: deny` in agent frontmatter to restrict them. See the [OpenCode agent specification](https://opencode.ai/docs) for the full permission model.

If OpenCode updates its agent frontmatter format, check the [OpenCode agent specification](https://opencode.ai/docs) to verify these files remain current.

> **Note for maintainers:** The agent body content is duplicated across three locations: `.opencode/agents/`, `agents/` (Claude Code), and `.github/agents/` (Copilot CLI). Any change to the review or test protocol must be applied in all three.

## Philosophy

- **One surgeon, many supporters.** Productivity comes from keeping the chief programmer focused, not from adding more equal contributors.
- **Roles are process gates, not suggestions.** When a role applies, invoking its skill is mandatory — it protects the quality of the Surgeon's work.
- **Delegate early, not late.** Recognizing the need for a supporting role at the start of a task is cheaper than discovering it after struggling.
- **Code belongs to the system.** Write as if a future Surgeon will read it cold.

## License

MIT License — see [LICENSE](LICENSE) for details.
