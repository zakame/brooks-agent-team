# brooks-agent-team

A Claude Code skills plugin that organizes AI-assisted software development around Fred Brooks' **Surgical Team** model from *The Mythical Man-Month* (1975).

Instead of every team member working on all parts of a system, the Surgical Team concentrates critical work in one skilled "surgeon" (chief programmer), supported by specialized roles that keep the surgeon focused and productive. This plugin maps those roles to Claude Code skills and agent dispatch templates.

## Inspiration

This project draws from two sources:

**Fred Brooks' Surgical Team** (Chapter 3, *The Mythical Man-Month*, 1975), originally conceived by Harlan Mills: a small, highly specialized team organized around a single chief programmer who writes all critical code, supported by a copilot, tester, toolsmith, editor, administrator, language lawyer, and program clerk — each with a distinct, non-overlapping responsibility.

**[Superpowers by Jesse Vincent](https://github.com/obra/superpowers)**: a Claude Code skills framework that demonstrated how composable, role-aware skills can guide an AI agent through disciplined software development workflows. The structure, conventions, and plugin format of this project follow Superpowers' design closely.

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

### For a single session

Start Claude Code with the `--plugin-dir` flag pointing to this repository:

```bash
claude --plugin-dir /path/to/brooks-agent-team
```

Skills are immediately available. To reload after making changes without restarting:

```
/reload-plugins
```

### Persistent use

Clone this repository to a stable location and add the `--plugin-dir` flag to your shell alias or Claude Code configuration:

```bash
git clone https://github.com/zakame/brooks-agent-team ~/.claude/plugins/brooks-agent-team
```

Then start Claude Code with:

```bash
claude --plugin-dir ~/.claude/plugins/brooks-agent-team
```

## Usage

### Assemble the team

Run the `/assemble-team` command at the start of any development session to get a project-contextual briefing on which roles apply and how to invoke them:

```
/assemble-team
```

Claude will survey your project and present a tailored overview of the team, with specific suggestions for each role based on what it finds.

### Invoke skills directly

Skills are namespaced under `brooks-agent-team:`. Invoke any role skill by name:

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

Once the plugin is loaded, Claude reads each skill's `description` field and automatically invokes the relevant skill when your request matches its trigger conditions. For example:

- Starting an implementation task → `surgeon` skill loads
- Completing a feature → `copilot` skill is suggested
- Spotting repeated manual operations → `toolsmith` skill activates
- Encountering a language edge case → `language-lawyer` skill is invoked

### Dispatch subagent roles

The **Copilot** and **Tester** roles can be dispatched as independent subagents, allowing the Surgeon to continue working on the critical path while review or test writing happens in parallel.

When the Copilot or Tester skill is invoked, it will prompt you to either work inline or dispatch a subagent using the templates in `agents/copilot.md` and `agents/tester.md`.

## Project Structure

```
.claude-plugin/
  plugin.json               Plugin manifest
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
agents/
  copilot.md                Dispatch template for Copilot subagent
  tester.md                 Dispatch template for Tester subagent
commands/
  assemble-team.md          /assemble-team slash command
```

## Philosophy

- **One surgeon, many supporters.** Productivity comes from keeping the chief programmer focused, not from adding more equal contributors.
- **Roles are process gates, not suggestions.** When a role applies, invoking its skill is mandatory — it protects the quality of the Surgeon's work.
- **Delegate early, not late.** Recognizing the need for a supporting role at the start of a task is cheaper than discovering it after struggling.
- **Code belongs to the system.** Write as if a future Surgeon will read it cold.

## License

MIT License — see [LICENSE](LICENSE) for details.
