# brooks-agent-team

A skills plugin that organizes AI-assisted software development around Fred Brooks' **Surgical Team** model from *The Mythical Man-Month* (1975). Compatible with **OpenAI Codex**, **GitHub Copilot CLI**, **Claude Code**, and **OpenCode**.

Instead of every team member working on all parts of a system, the Surgical Team concentrates critical work in one skilled "surgeon" (chief programmer), supported by specialized roles that keep the surgeon focused and productive. This plugin maps those roles to agent skills and dispatch templates.

## Inspiration

This project draws from two sources:

**Fred Brooks' Surgical Team** (Chapter 3, *The Mythical Man-Month*, 1975), originally conceived by Harlan Mills: a small, highly specialized team organized around a single chief programmer who writes all critical code, supported by a copilot, tester, toolsmith, editor, administrator, language lawyer, and program clerk — each with a distinct, non-overlapping responsibility.

**[Superpowers by Jesse Vincent](https://github.com/obra/superpowers)**: a Claude Code skills framework that demonstrated how composable, role-aware skills can guide an AI agent through disciplined software development workflows. The structure, conventions, and plugin format of this project follow Superpowers' design closely.

The `SKILL.md` format used here conforms to the [Agent Skills open standard](https://github.com/agentskills/agentskills), which is supported by GitHub Copilot CLI, Claude Code, OpenCode, and OpenAI Codex (via the `.agents/skills/` mirror).

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

### OpenAI Codex

#### Via marketplace (recommended)

Register the marketplace, then install `brooks-agent-team` from Codex's plugin UI:

```
codex plugin marketplace add zakame/skills-marketplace
```

After installation, the bundled skills are available through Codex's Skills layer. Marketplace installation covers skills; Codex custom subagents still need the `.codex/agents/` setup below. To update to a newer version:

```
codex plugin marketplace upgrade
```

#### Subagent configuration

Five roles are available as spawnable Codex subagents by their TOML `name` fields: `copilot`, `editor`, `language_lawyer`, `tester`, and `toolsmith`. The shared skill remains named `language-lawyer`. Codex discovers personal custom agents from `~/.codex/agents/`, so install the Brooks subagents there to use them across projects:

```bash
mkdir -p ~/.codex/agents
cp /path/to/brooks-agent-team/.codex/agents/*.toml ~/.codex/agents/
```

For a single project only, copy them into that project's `.codex/agents/` directory instead:

```bash
mkdir -p .codex
mkdir -p .codex/agents
cp /path/to/brooks-agent-team/.codex/agents/*.toml .codex/agents/
```

For session-wide subagent tunables (`max_threads`, `max_depth`), merge the `[features]` and `[agents]` settings from `.codex/config.toml` into `~/.codex/config.toml`. If you do not already have a user config, you can copy the provided one:

```bash
mkdir -p ~/.codex
cp /path/to/brooks-agent-team/.codex/config.toml ~/.codex/config.toml
```

For project-only tunables, copy the same file into `.codex/config.toml`:

```bash
mkdir -p .codex
cp /path/to/brooks-agent-team/.codex/config.toml .codex/config.toml
```

See the [Codex Subagents documentation](https://developers.openai.com/codex/subagents) for the full configuration schema.

Invoke the skills directly in a Codex session:

```
Use the assemble-team skill
Use the surgeon skill to implement the feature.
Use the copilot skill to review the completed diff.
```

### GitHub Copilot CLI

#### Via marketplace (recommended)

Register the marketplace, then install the plugin:

```
copilot plugin marketplace add zakame/skills-marketplace
copilot plugin install brooks-agent-team@zakame-skills-marketplace
```

Skills are available immediately. To update to a newer version:

```
copilot plugin update brooks-agent-team
```

#### Manual install (alternate)

To load skills directly from a local clone — useful when developing or testing changes to the skills themselves:

```bash
git clone https://github.com/zakame/brooks-agent-team /path/to/brooks-agent-team
```

Register the skills directory in a Copilot CLI session:

```
/skills add /path/to/brooks-agent-team/skills
```

#### Custom agents

The `copilot`, `tester`, and `language-lawyer` **custom agents** are bundled in `.github/agents/`. For marketplace installs, they are automatically available via `/agents` after `copilot plugin install` — no additional setup needed.

For manual installs, if the agents are not discovered automatically, copy them to `~/.copilot/agents/`:

```bash
cp /path/to/brooks-agent-team/.github/agents/*.agent.md ~/.copilot/agents/
```

See [Dispatch subagent roles](#dispatch-subagent-roles) for invocation examples.

### OpenCode

#### Skills

OpenCode discovers skills from `.opencode/skills/`, `.claude/skills/`, and `~/.config/opencode/skills/` (global). Clone this repository to a stable location of your choice, then symlink each skill subdirectory into a discovery path:

```bash
git clone https://github.com/zakame/brooks-agent-team /path/to/brooks-agent-team

# Project-level (one-time per project):
mkdir -p .opencode/skills
ln -sf /path/to/brooks-agent-team/skills/* .opencode/skills/

# OR global (available in all projects):
mkdir -p ~/.config/opencode/skills
ln -sf /path/to/brooks-agent-team/skills/* ~/.config/opencode/skills/
```

To update, run `git pull` inside `/path/to/brooks-agent-team`.

#### Subagent roles

OpenCode auto-discovers agent definitions from `.opencode/agents/` in your project directory. To make the Copilot, Tester, and Language Lawyer subagents available:

```bash
mkdir -p .opencode/agents
cp /path/to/brooks-agent-team/.opencode/agents/*.md .opencode/agents/
```

For cross-project use, global agent discovery paths are not yet standardized in OpenCode — check the [OpenCode agent documentation](https://opencode.ai/docs/agents/) for current options.

See [Dispatch subagent roles](#dispatch-subagent-roles) for invocation examples.

### Claude Code

#### Via marketplace (recommended)

Register the marketplace, then install the plugin:

```
/plugin marketplace add zakame/skills-marketplace
/plugin install brooks-agent-team@zakame-skills-marketplace
```

Skills are available immediately. To update to a newer version, run `/plugin install brooks-agent-team@zakame-skills-marketplace`.

#### Developer mode (alternate)

To load skills directly from a local clone — useful when developing or testing changes to the skills themselves:

```bash
git clone https://github.com/zakame/brooks-agent-team /path/to/brooks-agent-team
claude --plugin-dir /path/to/brooks-agent-team
```

To reload after making changes without restarting:

```
/reload-plugins
```

## Usage

### Two ways to start

| Skill / Command | Tool | When to use |
|---------|------|-------------|
| `assemble-team` skill | OpenAI Codex, Copilot CLI, Claude Code & OpenCode | Single-session work — one AI instance plays all roles sequentially |
| `/assemble-team` command | Claude Code only | Same as above, as a slash command |
| `assemble-with-fleet` skill | Copilot CLI & OpenCode | Parallel work — spawns one independent session per role (Copilot CLI uses `/fleet`; OpenCode uses the task tool) |
| `/assemble-with-agent-teams` command | Claude Code only | Parallel work — spawns via Claude Code Agent Teams |

### `assemble-team` — single-session briefing

Run at the start of any development session to get a project-contextual briefing on which roles apply and how to invoke them.

**Copilot CLI:**
```
Use the assemble-team skill
```

**OpenAI Codex:**
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

**OpenAI Codex** — uses Codex Subagents with per-role agent configs copied or symlinked into `~/.codex/agents/` for user-level installation, or into a project's `.codex/agents/` directory for project-only installation. Codex subagent workflows are enabled by default; this repo's `.codex/config.toml` sets only Brooks-specific tunables.
```
# The Surgeon spawns specialists by name during a session — for example:
Spawn the copilot subagent to review the authentication changes.
Spawn the tester subagent to write tests for the payment module.
Spawn the language_lawyer subagent to research this API deprecation.
```

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

**OpenAI Codex** — use the skill name directly in your prompt:
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

Subagent-capable roles can be dispatched as independent agents, allowing the Surgeon to continue working on the critical path while review, test writing, documentation, automation, or edge-case research happens in parallel. Codex exposes five Brooks subagents; Copilot CLI, Claude Code, and OpenCode expose Copilot, Tester, and Language Lawyer.

**OpenAI Codex** — uses the agent definitions after you copy or symlink them into `~/.codex/agents/` or a project `.codex/agents/` directory. Five roles are spawnable as subagents by TOML `name`: `copilot`, `editor`, `language_lawyer`, `tester`, and `toolsmith`. The shared Language Lawyer skill is still `language-lawyer`. The Copilot and Language Lawyer agents are sandboxed as read-only (`sandbox_mode = "read-only"`) to enforce their review-only roles; the other three (editor, tester, toolsmith) omit `sandbox_mode` so the default permissive sandbox applies. The Surgeon (main session), Administrator, and Program Clerk are inline-only — they exist as skills under `skills/<role>/SKILL.md` but are not exposed as Codex subagents.

**Copilot CLI** — use the custom agents in `.github/agents/` (copy to `~/.copilot/agents/` for cross-project use):
```
Use the copilot agent to review the authentication changes.
Use the tester agent to write tests for the payment module.
Use the language-lawyer agent to research this API deprecation.
```

**Claude Code** — use the dispatch templates in `agents/copilot.md`, `agents/tester.md`, and `agents/language-lawyer.md`.

**OpenCode** — use the agent definitions in `.opencode/agents/`:
```
Dispatch the Copilot agent to review the authentication changes.
Dispatch the Tester agent to write tests for the payment module.
Dispatch the Language Lawyer agent to research this API deprecation.
```

The Copilot agent is read-only (permissions deny `edit`, `bash`, and `webfetch`); the Tester agent can write files and run shell commands (permissions allow `edit` and `bash`); the Language Lawyer can fetch web resources and run shell commands (permissions allow `bash` and `webfetch`, deny `edit`).

## Repository Structure

```
.agents/skills/                 OpenAI Codex skill discovery mirror (symlinks to skills/)
.codex-plugin/                  OpenAI Codex plugin manifest
  plugin.json                     Plugin metadata and bundled skills path
.codex/                         OpenAI Codex configuration
  config.toml                     Subagent feature flag and session-wide tunables (max_threads, max_depth)
  agents/                         Per-role TOML configs for the 5 spawnable subagents
    copilot.toml                    Code reviewer agent (read-only sandbox)
    editor.toml                     Documentation agent
    language-lawyer.toml            Language/framework specialist agent
    tester.toml                     Adversarial tester agent
    toolsmith.toml                  Automation builder agent

skills/                         Shared across all platforms (Agent Skills standard)
  using-brooks-team/SKILL.md      Meta-skill: team orientation and role routing
  surgeon/SKILL.md                Chief programmer operating guide
  copilot/SKILL.md                Code review and backup
  tester/SKILL.md                 Adversarial test strategy
  administrator/SKILL.md          Task tracking and scope defense
    agents/openai.yaml            Codex Skills opt-out (inline-only role)
  editor/SKILL.md                 Documentation and accuracy
  toolsmith/SKILL.md              Custom tool builder
  language-lawyer/SKILL.md        Language and framework edge cases
  program-clerk/SKILL.md          Code organization and structure
    agents/openai.yaml            Codex Skills opt-out (inline-only role)
  assemble-team/SKILL.md          Team briefing skill (OpenAI Codex, Copilot CLI, Claude Code, and OpenCode)
  assemble-with-fleet/SKILL.md    Parallel team spawn via Copilot CLI fleet mode or OpenCode task tool

.claude-plugin/                 Claude Code specific
  plugin.json                     Plugin manifest (name, version, author)
agents/                         Claude Code dispatch templates
  copilot.md                      Subagent template for Copilot role
  tester.md                       Subagent template for Tester role
  language-lawyer.md              Subagent template for Language Lawyer role
commands/                       Claude Code slash commands
  assemble-team.md                /assemble-team command
  assemble-with-agent-teams.md    /assemble-with-agent-teams command

.github/agents/                 GitHub Copilot CLI custom agents
  copilot.agent.md                Code reviewer agent
  tester.agent.md                 Adversarial tester agent
  language-lawyer.agent.md        Language and framework edge-case agent

.opencode/agents/               OpenCode custom agents
  copilot.md                      Code reviewer agent
  tester.md                       Adversarial tester agent
  language-lawyer.md              Language and framework edge-case agent
```

### What lives where

The `skills/` directory is the core of the plugin. Each subdirectory contains a `SKILL.md` file conforming to the [Agent Skills open standard](https://github.com/agentskills/agentskills), which is supported by GitHub Copilot CLI, Claude Code, OpenCode, and OpenAI Codex (via the `.agents/skills/` mirror). These files work on any platform that reads the standard.

Platform-specific files provide deeper integration:

| Directory | Platform | Purpose |
|-----------|----------|---------|
| `.codex-plugin/` | OpenAI Codex | Plugin manifest for marketplace installs; exposes `skills/` as bundled Codex skills |
| `.agents/skills/` | OpenAI Codex | Skill discovery mirror — symlinks to the canonical `skills/` directories (Skills layer) |
| `.codex/config.toml` | OpenAI Codex | Subagent feature flag and session-wide tunables (`max_threads`, `max_depth`) |
| `.codex/agents/` | OpenAI Codex | Per-role TOML configs for 5 spawnable subagents (`copilot`, `editor`, `language_lawyer`, `tester`, `toolsmith`) |
| `skills/<role>/agents/openai.yaml` | OpenAI Codex | Per-skill Codex policy — used by `administrator` and `program-clerk` to opt out of implicit Skills-layer discovery |
| `.claude-plugin/` | Claude Code & Copilot CLI | Plugin manifest (marketplace loading) |
| `agents/` | Claude Code | Dispatch templates for subagent roles (Copilot, Tester, Language Lawyer) |
| `commands/` | Claude Code | Slash commands (`/assemble-team`, `/assemble-with-agent-teams`) |
| `.github/agents/` | Copilot CLI | Custom agent definitions for `/agents` dispatch (Copilot, Tester, Language Lawyer) |
| `.opencode/agents/` | OpenCode | Custom agent definitions for subagent dispatch (Copilot, Tester, Language Lawyer) |
| `skills/assemble-team/` | OpenAI Codex, Copilot CLI & OpenCode | Team briefing skill |
| `skills/assemble-with-fleet/` | Copilot CLI & OpenCode | Parallel spawn via fleet mode or task tool |

> **Note:** Copilot CLI recognizes `.claude-plugin/` in addition to `.github/plugin/` when loading plugin manifests.

### OpenCode compatibility

The eight role skills in `skills/` are compatible with [OpenCode](https://opencode.ai) via the [Agent Skills open standard](https://github.com/agentskills/agentskills) — OpenCode reads `SKILL.md` files the same way Claude Code and Copilot CLI do. See [OpenCode skills documentation](https://opencode.ai/docs/skills/) for the full discovery path list and `SKILL.md` format reference.

Full agent dispatch is supported through `.opencode/agents/`, which includes Copilot, Tester, and Language Lawyer subagent definitions. These files use OpenCode's [agent frontmatter format](https://opencode.ai/docs/agents/) (`description`, `mode: subagent`, `permission` block). Permission keys map to specific tools: `edit` controls write/edit/patch operations, `bash` controls shell commands, `webfetch` controls web fetches, and `read` controls file reads (allowed by default). The Copilot sets `edit: deny`, `bash: deny`, and `webfetch: deny`; the Tester sets `edit: allow` and `bash: allow`. To restrict file reads: `read: deny`.

If OpenCode updates its agent frontmatter format, check the [OpenCode agent specification](https://opencode.ai/docs/agents/) to verify these files remain current.

> **Note for maintainers:** The Copilot, Tester, and Language Lawyer agent body content is duplicated across all four locations: `.codex/agents/` (Codex), `.opencode/agents/` (OpenCode), `agents/` (Claude Code), and `.github/agents/` (Copilot CLI). Any change to those three protocols must be applied in all four. (The `.codex/agents/editor.toml` and `.codex/agents/toolsmith.toml` bodies are Codex-only.)

### OpenAI Codex compatibility

Codex marketplace installs use `.codex-plugin/plugin.json`, which exposes the shared `skills/` directory as bundled Codex skills. For local repo-scoped development, `.agents/skills/` mirrors those same canonical `skills/` directories with symlinks, so there is only one source of truth for each skill.

Codex exposes the Surgical Team across two complementary dispatch surfaces:

**1. Skills layer (implicit dispatch).** Codex matches user prompts to `SKILL.md` `description` fields from installed plugin skills or local `.agents/skills/` directories and can auto-load the matching skill. Six role skills (`surgeon`, `copilot`, `tester`, `editor`, `toolsmith`, `language-lawyer`) and two meta skills (`assemble-team`, `using-brooks-team`) are available for implicit invocation when their descriptions match the task. Two role skills — `administrator` and `program-clerk` — opt out by declaring `policy.allow_implicit_invocation: false` in `skills/<role>/agents/openai.yaml`. These two are inline-only by Brooks design: the Surgeon invokes them deliberately when project state needs to be made explicit or when codebase organization is in question, rather than letting them auto-trigger on incidental prompts.

**2. Subagents layer (explicit dispatch).** `.codex/agents/*.toml` declares five spawnable subagents — `copilot`, `editor`, `language_lawyer`, `tester`, and `toolsmith` — that the Surgeon explicitly spawns for parallel work after those TOML files are present in `~/.codex/agents/` or the active project's `.codex/agents/` directory. Each TOML carries a `name`, a `description`, an optional `sandbox_mode`, and a `developer_instructions` body. The Copilot and Language Lawyer agents use `sandbox_mode = "read-only"` to enforce their review-only roles; the other three (editor, tester, toolsmith) omit `sandbox_mode` so the default permissive sandbox applies. Each `developer_instructions` body opens with a `[CODEX-STOP]` block listing `surgeon`, `using-brooks-team`, `assemble-team`, and `assemble-with-fleet` as forbidden re-entry targets, mirroring the `<SUBAGENT-STOP>` pattern used in Claude Code and OpenCode agent files. Agents are auto-discovered from `.codex/agents/` by directory scan — `.codex/config.toml` itself only sets session-wide tunables (`max_threads = 8`, `max_depth = 1`) and re-asserts the stable `multi_agent` feature flag as a guard against user- or system-level overrides. The Surgeon (main session), Administrator, and Program Clerk are not exposed as subagents — they are skill-only roles.

Codex extends the subagent dispatch surface to five roles (vs. three on Claude Code, OpenCode, and Copilot CLI, which expose only Copilot, Tester, and Language Lawyer as subagents). Codex's per-agent TOML schema with rich `developer_instructions` makes role-specific subagents like Editor and Toolsmith useful on Codex even though they remain inline-only on the other platforms.

The single-session workflow (`assemble-team`) is portable to Codex without modification — it loads through the Skills layer alongside the other implicit skills.

If Codex updates its agent TOML schema or Skills policy keys, check the [Codex Subagents documentation](https://developers.openai.com/codex/subagents) and the [Codex Skills documentation](https://developers.openai.com/codex/skills) to verify these files remain current.

## Philosophy

- **One surgeon, many supporters.** Productivity comes from keeping the chief programmer focused, not from adding more equal contributors.
- **Roles are process gates, not suggestions.** When a role applies, invoking its skill is mandatory — it protects the quality of the Surgeon's work.
- **Delegate early, not late.** Recognizing the need for a supporting role at the start of a task is cheaper than discovering it after struggling.
- **Code belongs to the system.** Write as if a future Surgeon will read it cold.

## License

MIT License — see [LICENSE](LICENSE) for details.
