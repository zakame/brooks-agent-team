# brooks-agent-team

A skills plugin that organizes AI-assisted software development around Fred Brooks' **Surgical Team** model from *The Mythical Man-Month* (1975). Compatible with **OpenAI Codex**, **GitHub Copilot CLI**, **Claude Code**, and **OpenCode**.

Instead of every team member working on all parts of a system, the Surgical Team concentrates critical work in one skilled "surgeon" (chief programmer), supported by specialized roles that keep the surgeon focused and productive. This plugin maps those roles to agent skills and dispatch templates.

## Inspiration

This project draws from two sources:

**Fred Brooks' Surgical Team** (Chapter 3, *The Mythical Man-Month*, 1975), originally conceived by Harlan Mills: a small, highly specialized team organized around a single chief programmer who writes all critical code, supported by a copilot, tester, toolsmith, editor, administrator, language lawyer, and program clerk — each with a distinct, non-overlapping responsibility. Brooks' case for this structure is conceptual integrity: a system designed by one mind (or a few, tightly coordinated) is more coherent than one designed by a large, egalitarian team, even if it takes longer to build. Brooks' original roster also includes two secretaries, one for the administrator and one for the editor. This plugin omits them since they existed to handle human clerical work (correspondence, filing, scheduling) that an AI agent doesn't need.

**[Superpowers by Jesse Vincent](https://github.com/obra/superpowers)**: a Claude Code skills framework that demonstrated how composable, role-aware skills can guide an AI agent through disciplined software development workflows. The structure, conventions, and plugin format of this project follow Superpowers' design closely.

The `SKILL.md` format used here conforms to the [Agent Skills open standard](https://github.com/agentskills/agentskills), which is supported by GitHub Copilot CLI, Claude Code, OpenCode, and OpenAI Codex (via the `.agents/skills/` mirror described below).

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

Codex uses `AGENTS.md` for persistent repository guidance. This repository includes an `AGENTS.md` file plus a `.agents/skills/` mirror of the canonical `skills/` directory for repo-local Codex workflows.

Skills are discovered via `.agents/skills/` when Codex is launched inside this repository. To use these skills from other projects, install them into Codex's user-level skill path:

```bash
# Global skills, available from any project
mkdir -p ~/.agents/skills
ln -sfn /path/to/brooks-agent-team/skills/* ~/.agents/skills/
```

For project-local use, copy or symlink the skills into the target repository instead:

```bash
mkdir -p .agents/skills
ln -sfn /path/to/brooks-agent-team/skills/* .agents/skills/
```

For native Codex subagents (enabling `/agent` switching in Codex CLI), this repository includes standalone TOML configs for the seven specialist roles (Copilot, Tester, Editor, Toolsmith, Language Lawyer, Program Clerk, Administrator) in `.codex/agents/`. The Surgeon is the default chief-programmer role in any Codex session and is not a dispatchable Codex agent. Install the specialist agents globally or project-locally:

```bash
# Global agents, available from any project
mkdir -p ~/.codex/agents
ln -sf /path/to/brooks-agent-team/.codex/agents/*.toml ~/.codex/agents/

# Or project-local agents
mkdir -p .codex/agents
cp /path/to/brooks-agent-team/.codex/agents/*.toml .codex/agents/
```

The optional `.codex/config.toml` in this repository only sets subagent runtime limits such as `agents.max_concurrent_threads_per_session`; merge those settings into your own `.codex/config.toml` if you want them. Project-scoped `.codex/` configuration may require the project to be trusted before Codex loads it. See the [Codex configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference) for the full configuration schema.

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

Spawn one independent AI session per role so that Copilot reviews, Tester writes tests, and Language Lawyer researches edge cases while you continue on the critical path.

**OpenAI Codex** — uses Codex multi-agent mode with standalone per-role agent configs in `.codex/agents/` or `~/.codex/agents/`. Multi-agent is stable and enabled by default in current Codex CLI releases. Each of the seven specialist roles (Copilot, Tester, Editor, Toolsmith, Language Lawyer, Program Clerk, Administrator) is defined by its matching TOML file.
```
# Codex discovers agents from .codex/agents/ or ~/.codex/agents/
# Explicitly ask Codex to spawn specialists; use /agent to inspect or steer threads
Spawn a copilot agent to review the current diff, a tester agent to audit test gaps,
and a language-lawyer agent to check version-sensitive API claims. Wait for all
three, then summarize their findings by role.
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

**What you get by platform:**
- **Codex:** explicit parallel specialist subagents, consolidated summaries back to the main thread, and `/agent` for inspecting, steering, and switching between agent threads. Codex does not currently provide a Claude Agent Teams-style shared task list or direct teammate mailbox.
- **Claude Agent Teams:** independent teammates coordinated by a lead, with a shared task list, dependency tracking, direct teammate messaging, and file ownership guidance.
- **Copilot CLI / OpenCode:** fleet/task-tool style parallel workers following the shared task list included in their prompts or platform session state.

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

The **Copilot**, **Tester**, and **Language Lawyer** roles can be dispatched as independent subagents, allowing the Surgeon to continue working on the critical path while review, test writing, or language/framework research happens in parallel.

**OpenAI Codex** — all seven specialist roles are standalone TOML files in `.codex/agents/` for project use or `~/.codex/agents/` for global use; the Surgeon is the default orchestrator and has no agent file in `.codex/agents/` — it is not a dispatch target. Copilot, Administrator, and Program Clerk set `sandbox_mode = "read-only"`; the Language Lawyer inherits the parent session's sandbox and approval policy and has a `[CODEX-STOP]` instruction guard discouraging file edits (prompt-level convention, not a sandbox enforcement). Subagent dispatch is always explicit.

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
Dispatch the Language Lawyer agent to research this framework edge case.
```

On Claude Code, Copilot CLI, and OpenCode, the Copilot agent is read-only (permissions deny `edit`, `bash`, and `webfetch`); the Tester agent can write files and run shell commands (permissions allow `edit` and `bash`); the Language Lawyer can search the web, fetch URLs, and run shell commands (permissions allow `bash`, `webfetch`, and `websearch`, deny `edit`). Codex agents use Codex sandbox and approval settings instead.

## Repository Structure

```
AGENTS.md                       OpenAI Codex repository instructions and role routing
.agents/skills/                 OpenAI Codex skill discovery mirror (symlinks to skills/)
.codex/                         OpenAI Codex configuration
  config.toml                     Optional subagent runtime limits
  agents/                         Per-role specialist TOML configs
    copilot.toml                    Code reviewer agent (read-only sandbox)
    tester.toml                     Adversarial tester agent
    editor.toml                     Documentation agent
    toolsmith.toml                  Automation builder agent
    language-lawyer.toml            Language/framework specialist agent
    program-clerk.toml              Code organization agent
    administrator.toml              Task tracking agent

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
  assemble-team/SKILL.md          Team briefing skill (all platforms)
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
| `AGENTS.md` | OpenAI Codex | Repository-level instructions and role-routing guidance |
| `.agents/skills/` | OpenAI Codex | Project-local mirror of the canonical `skills/` directory |
| `.codex/config.toml` | OpenAI Codex | Optional subagent runtime limits such as `agents.max_concurrent_threads_per_session` |
| `.codex/agents/` | OpenAI Codex | Per-role TOML agent configs with developer instructions |
| `.claude-plugin/` | Claude Code & Copilot CLI | Plugin manifest (marketplace loading) |
| `agents/` | Claude Code | Dispatch templates for subagent roles (Copilot, Tester) |
| `commands/` | Claude Code | Slash commands (`/assemble-team`, `/assemble-with-agent-teams`) |
| `.github/agents/` | Copilot CLI | Custom agent definitions for `/agent` dispatch |
| `.opencode/agents/` | OpenCode | Custom agent definitions for subagent dispatch |
| `skills/assemble-team/` | All platforms (Claude Code, OpenAI Codex, Copilot CLI & OpenCode) | Team briefing skill |
| `skills/assemble-with-fleet/` | Copilot CLI & OpenCode | Parallel spawn via fleet mode or task tool |

> **Note:** Copilot CLI recognizes `.claude-plugin/` in addition to `.github/plugin/` when loading plugin manifests.

### OpenCode compatibility

The eight role skills in `skills/` are compatible with [OpenCode](https://opencode.ai) via the [Agent Skills open standard](https://github.com/agentskills/agentskills) — OpenCode reads `SKILL.md` files the same way Claude Code and Copilot CLI do. See [OpenCode skills documentation](https://opencode.ai/docs/skills/) for the full discovery path list and `SKILL.md` format reference.

Full agent dispatch is supported through `.opencode/agents/`, which includes Copilot, Tester, and Language Lawyer subagent definitions. These files use OpenCode's [agent frontmatter format](https://opencode.ai/docs/agents/) (`description`, `mode: subagent`, `permission` block). Permission keys map to specific tools: `edit` controls write/edit/patch operations, `bash` controls shell commands, `webfetch` controls web fetches, and `read` controls file reads (allowed by default). The Copilot sets `edit: deny`, `bash: deny`, and `webfetch: deny`; the Tester sets `edit: allow` and `bash: allow`. To restrict file reads: `read: deny`.

If OpenCode updates its agent frontmatter format, check the [OpenCode agent specification](https://opencode.ai/docs/agents/) to verify these files remain current.

> **Note for maintainers:** The core dispatch set (Copilot, Tester, Language Lawyer) is duplicated across four locations: `.codex/agents/` (Codex), `.opencode/agents/` (OpenCode), `agents/` (Claude Code), and `.github/agents/` (Copilot CLI). The four Codex-only specialists (Editor, Toolsmith, Program Clerk, Administrator) exist only in `.codex/agents/` and have no peer files on other platforms. This has already caused real drift (a permission-grant fix landed with three different wordings; a skill exclusion was added to two copies and missed a third). When changing a role's dispatch template, update every platform copy that role has and check that these stay equivalent:
> - The protocol body (review steps, failure-mode checklist, output format)
> - The skill-exclusion list (`SUBAGENT-STOP` / `[CODEX-STOP]`)
> - The tool/permission grant, expressed in each platform's own format (Claude Code `tools`/`disallowedTools`, OpenCode `permission`, Copilot CLI `tools`, Codex `sandbox_mode`)
>
> Now that a fourth platform is here, a single canonical source per role with generated or symlinked platform shims is worth serious consideration instead of continuing to hand-sync four copies.

### OpenAI Codex compatibility

[OpenAI Codex](https://learn.chatgpt.com/docs/codex/cli) reads `AGENTS.md` for persistent repository guidance. This repository includes `AGENTS.md` plus a `.agents/skills/` mirror implemented as symlinks to the canonical `skills/` directories, so there is only one source of truth for each repo-local skill.

Codex multi-agent support discovers standalone custom agent files from `.codex/agents/*.toml` for project-scoped agents and `~/.codex/agents/*.toml` for global personal agents. The read-only roles — Copilot, Administrator, and Program Clerk — set `sandbox_mode = "read-only"` to enforce their non-editing roles. The Language Lawyer omits `sandbox_mode` (inheriting the parent session's sandbox and approval policy) and has a `[CODEX-STOP]` instruction guard discouraging file edits — this is a prompt-level convention, not a sandbox enforcement. Codex has no per-agent tool grants, so the Language Lawyer's research also depends on the parent session: enable web search in your Codex session for the role to verify claims against live documentation. The remaining write-capable roles (Tester, Editor, Toolsmith) also omit `sandbox_mode`. Each agent config includes `name`, `description`, and `developer_instructions` fields required by the current Codex TOML schema and a `[CODEX-STOP]` block discouraging re-entrant skill invocation, mirroring the `[SUBAGENT-STOP]` pattern used in Claude Code and OpenCode agent files.

Multi-agent mode is stable and enabled by default in current Codex CLI releases. No configuration changes are needed.

The single-session workflow (`assemble-team`) is portable to Codex without modification. When Codex runs inside this repository, the eight role skills plus `assemble-team` and `using-brooks-team` are available via the `.agents/skills/` mirror. The `assemble-with-fleet` skill is present for Copilot CLI and OpenCode compatibility, but Codex parallel work should use explicit custom-agent dispatch instead. For other projects, install the skills into that project's `.agents/skills/` directory or into the user-level `~/.agents/skills/` directory.

**Known limitation — explicit subagent dispatch.** Codex subagent dispatch is explicit: specialists must be spawned by prompting the orchestrator to delegate to them. The `/agent` command switches between active agent threads for inspection and follow-up. Codex skills can still be invoked explicitly or selected implicitly when the task matches the skill `description`; subagents are separate from skill activation. This means the Surgeon must consciously direct multi-agent team choreography.

> **Note:** "Implicit invocation" for skills means the agent may choose a skill when the task matches its `description`. It does not mean Codex will automatically spawn a specialist subagent.

Codex discovers `AGENTS.md` files by walking from the git root to the current working directory and concatenating them root-to-cwd (files closer to cwd appear later and win on conflicts). A global `~/.codex/AGENTS.override.md` suppresses `~/.codex/AGENTS.md`; at each project directory, `AGENTS.override.md` suppresses `AGENTS.md`, and `AGENTS.md` is used when neither override is present. The combined size of all included instruction files is capped at 32 KiB (`project_doc_max_bytes`); Codex stops including further files once that cap is reached.

If Codex updates its agent TOML schema or `AGENTS.md` conventions, check the [Codex subagent documentation](https://learn.chatgpt.com/docs/agent-configuration/subagents) to verify these files remain current.

## Philosophy

- **One surgeon, many supporters.** Productivity comes from keeping the chief programmer focused, not from adding more equal contributors.
- **Roles are process gates, not suggestions.** When a role applies, invoking its skill is mandatory — it protects the quality of the Surgeon's work.
- **Delegate early, not late.** Recognizing the need for a supporting role at the start of a task is cheaper than discovering it after struggling.
- **Code belongs to the system.** Write as if a future Surgeon will read it cold.

## License

MIT License — see [LICENSE](LICENSE) for details.
