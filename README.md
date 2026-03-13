# brooks-agent-team

A skills plugin that organizes AI-assisted software development around Fred Brooks' **Surgical Team** model from *The Mythical Man-Month* (1975). Compatible with **GitHub Copilot CLI** and **Claude Code**.

Instead of every team member working on all parts of a system, the Surgical Team concentrates critical work in one skilled "surgeon" (chief programmer), supported by specialized roles that keep the surgeon focused and productive. This plugin maps those roles to agent skills and dispatch templates.

## Inspiration

This project draws from two sources:

**Fred Brooks' Surgical Team** (Chapter 3, *The Mythical Man-Month*, 1975), originally conceived by Harlan Mills: a small, highly specialized team organized around a single chief programmer who writes all critical code, supported by a copilot, tester, toolsmith, editor, administrator, language lawyer, and program clerk — each with a distinct, non-overlapping responsibility.

**[Superpowers by Jesse Vincent](https://github.com/obra/superpowers)**: a Claude Code skills framework that demonstrated how composable, role-aware skills can guide an AI agent through disciplined software development workflows. The structure, conventions, and plugin format of this project follow Superpowers' design closely.

The `SKILL.md` format used here conforms to the [Agent Skills open standard](https://github.com/agentskills/agentskills), which is supported by both GitHub Copilot CLI and Claude Code.

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
| `assemble-team` skill | Copilot CLI & Claude Code | Single-session work — one AI instance plays all roles sequentially |
| `/assemble-team` command | Claude Code only | Same as above, as a slash command |
| `assemble-with-fleet` skill | Copilot CLI | Parallel work — spawns one independent session per role via `/fleet` |
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

The AI surveys your project and presents a tailored overview of the team. Roles are invoked on demand as the work requires them. Lightweight and works without any additional setup.

### Parallel team spawn

Spawn one independent AI session per role so that Copilot reviews and Tester writes tests while you continue on the critical path.

**Copilot CLI** — uses `assemble-with-fleet` skill (requires experimental fleet mode):
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

## Project Structure

```
.claude-plugin/
  plugin.json               Claude Code plugin manifest
.github/
  agents/
    copilot.agent.md        Copilot CLI custom agent: code reviewer
    tester.agent.md         Copilot CLI custom agent: adversarial tester
skills/
  using-brooks-team/        Meta-skill: team orientation and role routing
  surgeon/                  Chief programmer operating guide
  copilot/                  Code review and backup
  tester/                   Adversarial test strategy
  administrator/            Task tracking and scope defense
  editor/                   Documentation and accuracy
  toolsmith/                Custom tool builder
  language-lawyer/          Language and framework edge cases
  program-clerk/            Code organization and structure
  assemble-team/            Copilot CLI: team briefing skill
  assemble-with-fleet/      Copilot CLI: parallel team spawn via fleet mode
agents/
  copilot.md                Claude Code dispatch template for Copilot subagent
  tester.md                 Claude Code dispatch template for Tester subagent
commands/
  assemble-team.md          Claude Code: /assemble-team slash command
  assemble-with-agent-teams.md  Claude Code: /assemble-with-agent-teams slash command
```

## Philosophy

- **One surgeon, many supporters.** Productivity comes from keeping the chief programmer focused, not from adding more equal contributors.
- **Roles are process gates, not suggestions.** When a role applies, invoking its skill is mandatory — it protects the quality of the Surgeon's work.
- **Delegate early, not late.** Recognizing the need for a supporting role at the start of a task is cheaper than discovering it after struggling.
- **Code belongs to the system.** Write as if a future Surgeon will read it cold.

## License

MIT License — see [LICENSE](LICENSE) for details.
