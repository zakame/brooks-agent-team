# brooks-agent-team

A skills plugin that organizes AI-assisted software development around Fred Brooks' **Surgical Team** model from *The Mythical Man-Month* (1975). Compatible with **Claude Code**, **GitHub Copilot CLI**, **OpenCode**, **OpenAI Codex**, and **Grok Build**.

Instead of every team member working on all parts of a system, the Surgical Team concentrates critical work in one skilled "surgeon" (chief programmer), supported by specialized roles that keep the surgeon focused and productive. This plugin maps those roles to agent skills and dispatch templates.

## Contents

- [Inspiration](#inspiration)
- [The Team](#the-team)
- [Installation](#installation)
- [Usage](#usage)
- [Repository Structure](#repository-structure)
- [Philosophy](#philosophy)
- [License](#license)

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

### Grok Build

Grok Build has moved fast: this integration was originally written against 0.2.43 (2026-06), revised against 1.0.3 (2026-08-14), and re-verified against 1.0.13 (2026-09-01, the current release). All claims below still held as of 1.0.13 except `capability_mode`, which turned out to never have been a real `spawn_subagent` parameter (see the caveat under "Running the parallel team safely" below). If you're on a materially newer Grok Build release, re-check this section before trusting it.

#### Via Claude Code plugin compatibility (recommended)

Grok Build automatically reads Claude Code marketplaces, plugins, and skills via the `[compat.claude]` config block (`skills`, `rules`, `agents`, `mcps`, `hooks`, `sessions` — each defaults to `true` in `~/.grok/config.toml`). If you have installed `brooks-agent-team` as a Claude Code plugin (see the [Claude Code section](#claude-code) above), Grok picks it up with no extra setup — `grok inspect` lists the skills under `plugin: brooks-agent-team [claude]` and the dispatch agents under `plugin: brooks-agent-team` (verified on Grok 1.0.3). Note: `[compat.claude].agents` only gates `~/.claude/`-scoped instruction files and project `CLAUDE*.md`, not the `agents/` discovery below — that's separate, general plugin-component discovery.

Note that this loads the **released plugin version** from your Claude Code install. To pick up newer skills before a release, use the development setup below.

#### Local clone (development / testing)

**If you already have `brooks-agent-team` installed as a Claude Code plugin** (the recommended path above), Grok auto-discovers it via Claude-plugin compatibility, and that install will silently take priority over most other ways of pointing Grok at a local clone (verified live, 2026-09-01; see the two caveats below). The one method confirmed to reliably override it is a project-scoped plugin symlink:

```bash
git clone https://github.com/zakame/brooks-agent-team /path/to/brooks-agent-team

mkdir -p .grok/plugins
ln -sf /path/to/brooks-agent-team .grok/plugins/brooks-agent-team
```

On the Grok Build version tested (1.0.3), `.grok/plugins/` was observed to take priority over the Claude-plugin-compatibility layer, cleanly replacing the installed version's skills and agents with the clone's, without needing to uninstall anything. `grok inspect` will show the skills as `plugin: brooks-agent-team` with no `[claude]` tag, and file paths pointing at the clone. First use in a new project may show the plugin as untrusted; that only affects hooks and MCP servers (neither of which this plugin uses), not skill discovery. This discovery-order behavior isn't spelled out in Grok's public docs as a guaranteed contract, so re-verify with `grok inspect` if it stops working on a newer Grok release.

The following two methods also work, but each was observed to fail in its own way if `brooks-agent-team` is also installed via Claude Code (now, or later if you install it that way after setting one of these up). The project-scoped method above avoids both failure modes, so prefer it whenever a competing install might exist:

- Register the clone as a plugin path in `~/.grok/config.toml`. This also requires explicitly enabling it, and was observed to lose silently to any competing install of the same plugin name, including a Claude-plugin-compat one:

  ```toml
  [plugins]
  paths = ["/path/to/brooks-agent-team"]
  enabled = ["brooks-agent-team"]
  ```

- Or symlink the skills directly for project use. This scans as generic project skills, additive to (not replacing) any plugin-sourced skills of the same name, so a skill available from both sources will appear twice in `grok inspect` (once `type: project`, once `type: plugin`) if brooks-agent-team is also installed as a plugin:

  ```bash
  mkdir -p .grok/skills
  ln -sf /path/to/brooks-agent-team/skills/* .grok/skills/
  ```

Run `grok inspect` to confirm what Grok discovered for the current directory. Check for a stray `[claude]` tag (stale install still winning) or a duplicate skill name (both installs loaded side by side) before trusting local changes are actually in effect. To update a clone: `git pull` inside it.

#### Agent discovery

`.grok/agents/` discovery is documented in Grok's subagent docs: project-scope discovery is confirmed working — with your CWD inside the cloned repo, `grok inspect` lists `brooks-copilot`, `brooks-tester`, and `brooks-language-lawyer` as `project` agents (verified on Grok 1.0.3). Copy them to `~/.grok/agents/` for cross-project use. As of 1.0.3, agent frontmatter supports a real `tools:` field (restricts the subagent's available tools) and `mcpInheritance:` (controls which parent MCP servers a spawned subagent inherits) — `color:`/`model:` remain Claude-Code-only and still don't apply to Grok agents.

**Current Grok subagent spawning notes (2026-08-14, verified against Grok 1.0.3):**

The `assemble-with-grok-team` skill spawns each teammate directly with its full task in one call — no lightweight-first-turn/resume split:
- Copilot, Tester, and Language Lawyer are spawned directly by `subagent_type` (`brooks-copilot`, `brooks-tester`, `brooks-language-lawyer`) — their `.grok/agents/brooks-*.md` body is the role contract. Editor/Toolsmith/Program Clerk have no native Grok agent definition, so they spawn via `general-purpose` with an explicit skill-invocation instruction instead.
- This follows live headless smoke testing confirming Grok 1.0.3 runs named custom agents fully autonomously through multi-tool-call tasks (2-4 tool calls each, scaling with task size) — the single-turn limitation observed on Grok 0.2.43, which used to force a `general-purpose` + `resume_from` workaround, is resolved.
- The Surgeon monitors via `get_command_or_subagent_output` (Ctrl+T for todos, Ctrl+G for tasks, queue pane for background status); `resume_from` remains available for a genuine follow-up round after a teammate's task completes, but isn't needed for the first result.

**Running the parallel team safely:**
- Start via the `assemble-with-grok-team` skill — it sets up the shared `todo_write` list (the team's single source of truth, visible via Ctrl+T), the right `isolation` per role, and the file ownership rules. **Caveat (corrected 2026-09-01):** `capability_mode` is not a real `spawn_subagent` parameter, confirmed against current Grok documentation; an earlier round of this integration mistakenly treated it as one. `isolation: "worktree"` is enforced and is the real safety boundary for writers. For read-only roles, `.grok/agents/brooks-copilot.md`'s `tools:` frontmatter achieves a genuine hard write-block (confirmed); `brooks-language-lawyer.md`'s does not fully close the gap, since its required Bash access reopens a write-capable tool as a side effect — its "never edit" behavior still relies on its own prompt contract.
- Monitor with **Ctrl+T** (shared todos), **Ctrl+G** (tasks + background subagents), and **Ctrl+;** (queue / prompt status).
- Writing roles (Tester, Editor, Toolsmith, Program Clerk) work in isolated worktrees — their changes don't touch your workspace until you apply them. **Review before applying:** list worktrees with `x.ai/git/worktree/list` (or use the path the teammate reported), read the full diff with `git -C <worktree-path> diff HEAD`, then `x.ai/git/worktree/apply <id>` and check `git status`. If the `x.ai/git/worktree/*` extensions aren't in your tool list, fall back to manual git inspection and copying only the role's owned paths.
- Run one writing teammate at a time unless their ownership areas are provably disjoint — the skill includes a Surgeon checklist for this.
- Capture subagent IDs from each spawn result; you'll need them for `resume_from` follow-ups or `kill_command_or_subagent`.

For the full worktree review/apply workflow, spawn templates for every role, and safety rules, see `skills/assemble-with-grok-team/SKILL.md`.

## Usage

### Two ways to start

| Skill / Command | Tool | When to use |
|---------|------|-------------|
| `assemble-team` skill | All platforms (Claude Code, Copilot CLI, OpenCode, OpenAI Codex & Grok Build) | Single-session work — one AI instance plays all roles sequentially |
| `/assemble-team` command | Claude Code only | Same as above, as a slash command |
| `assemble-with-grok-team` skill | Grok Build only | Parallel work — spawns one independent subagent per role using Grok's native `spawn_subagent` + worktrees + shared `todo_write` list |
| `assemble-with-fleet` skill | Copilot CLI & OpenCode | Parallel work — spawns one independent session per role (Copilot CLI uses `/fleet`; OpenCode uses the task tool) |
| `/assemble-with-agent-teams` command | Claude Code only | Parallel work — spawns via Claude Code Agent Teams |

### `assemble-team` — single-session briefing

Run at the start of any development session to get a project-contextual briefing on which roles apply and how to invoke them.

**Claude Code:**
```
/assemble-team
```

**Copilot CLI:**
```
Use the assemble-team skill
```

**OpenCode:**
```
Use the assemble-team skill
```

**OpenAI Codex:**
```
Use the assemble-team skill
```

The AI surveys your project and presents a tailored overview of the team. Roles are invoked on demand as the work requires them. Lightweight and works without any additional setup.

### Parallel team spawn

Spawn one independent AI session per role so that Copilot reviews, Tester writes tests, and Language Lawyer researches edge cases while you continue on the critical path.

**Claude Code** — uses `/assemble-with-agent-teams` (requires [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams.md)):

Enable Agent Teams first. If you're on a current-generation model (Sonnet 5, Opus 4.8,
Fable 5, Mythos 5, or later), also opt back into the Task tools — as of Claude Code
v2.1.233 they're off by default for those models, and without them teammates fall back
to `SendMessage`-only coordination instead of the shared task list:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_ENABLE_TODO_TOOLS": "1"
  }
}
```
Then run:
```
/assemble-with-agent-teams
```

**Copilot CLI** — uses `assemble-with-fleet` skill (requires experimental fleet mode):
```
Use the assemble-with-fleet skill
```

**OpenCode** — uses `assemble-with-fleet` skill (spawns subagents via the task tool):
```
Use the assemble-with-fleet skill
```

**OpenAI Codex** — uses Codex multi-agent mode with standalone per-role agent configs in `.codex/agents/` or `~/.codex/agents/`. Multi-agent is stable and enabled by default in current Codex CLI releases. Each of the seven specialist roles (Copilot, Tester, Editor, Toolsmith, Language Lawyer, Program Clerk, Administrator) is defined by its matching TOML file.
```
# Codex discovers agents from .codex/agents/ or ~/.codex/agents/
# Explicitly ask Codex to spawn specialists; use /agent to inspect or steer threads
Spawn a copilot agent to review the current diff, a tester agent to audit test gaps,
and a language-lawyer agent to check version-sensitive API claims. Wait for all
three, then summarize their findings by role.
```

**Grok Build** — uses `assemble-with-grok-team` skill (native `spawn_subagent` dispatching named `brooks-*` agents directly with their full task, worktree isolation for writers, shared `todo_write` via Ctrl+T):
```
Use the assemble-with-grok-team skill
```

(Note: this skill was originally built around an observed single-turn limit on custom named agents in Grok 0.2.43, forcing a `general-purpose` + `resume_from` workaround. Live smoke testing confirmed that limitation is resolved in Grok 1.0.3, and the skill now spawns named agents directly.)

**What you get by platform:**
- **Claude Agent Teams:** independent teammates coordinated by a lead, with a shared task list, dependency tracking, direct teammate messaging, and file ownership guidance. The shared task list requires the Task tools (`TodoWrite`, `TaskCreate`, `TaskGet`, `TaskUpdate`, `TaskList`); those are on by default only on older models like Opus 4.7 — Sonnet 5, Opus 4.8, and later need `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` (see above), or teammates silently coordinate via messages only.
- **Copilot CLI / OpenCode:** fleet/task-tool style parallel workers following the shared task list included in their prompts or platform session state.
- **Codex:** explicit parallel specialist subagents, consolidated summaries back to the main thread, and `/agent` for inspecting, steering, and switching between agent threads. Codex does not currently provide a Claude Agent Teams-style shared task list or direct teammate mailbox.

### Invoke skills directly

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

**Copilot CLI** — use the skill name in your prompt:
```
Use the surgeon skill to start implementing this feature.
Use the copilot skill to review my changes.
Use the language-lawyer skill for this edge case.
```

**OpenCode** — use the skill name directly in a session prompt:
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

### Automatic invocation

The AI reads each skill's `description` field and automatically invokes the relevant skill when your request matches its trigger conditions. For example:

- Starting an implementation task → `surgeon` skill loads
- Completing a feature → `copilot` skill is suggested
- Spotting repeated manual operations → `toolsmith` skill activates
- Encountering a language edge case → `language-lawyer` skill is invoked

### Dispatch subagent roles

The **Copilot**, **Tester**, and **Language Lawyer** roles can be dispatched as independent subagents, allowing the Surgeon to continue working on the critical path while review, test writing, or language/framework research happens in parallel.

**Claude Code** — use the dispatch templates in `agents/copilot.md`, `agents/tester.md`, and `agents/language-lawyer.md`.

**Copilot CLI** — use the custom agents in `.github/agents/` (copy to `~/.copilot/agents/` for cross-project use):
```
Use the copilot agent to review the authentication changes.
Use the tester agent to write tests for the payment module.
Use the language-lawyer agent to research this API deprecation.
```

**OpenCode** — use the agent definitions in `.opencode/agents/`:
```
Dispatch the Copilot agent to review the authentication changes.
Dispatch the Tester agent to write tests for the payment module.
Dispatch the Language Lawyer agent to research this framework edge case.
```

On Claude Code, Copilot CLI, and OpenCode, the Copilot agent is read-only (permissions deny `edit`, `bash`, and `webfetch`); the Tester agent can write files and run shell commands (permissions allow `edit` and `bash`); the Language Lawyer can search the web, fetch URLs, and run shell commands (permissions allow `bash`, `webfetch`, and `websearch`, deny `edit`). Codex agents use Codex sandbox and approval settings instead.

**OpenAI Codex** — all seven specialist roles are standalone TOML files in `.codex/agents/` for project use or `~/.codex/agents/` for global use; the Surgeon is the default orchestrator and has no agent file in `.codex/agents/` — it is not a dispatch target. Copilot, Administrator, and Program Clerk set `sandbox_mode = "read-only"`; the Language Lawyer inherits the parent session's sandbox and approval policy and has a `[CODEX-STOP]` instruction guard discouraging file edits (prompt-level convention, not a sandbox enforcement). Subagent dispatch is always explicit.

**Grok Build** — custom agents live in `.grok/agents/` (`brooks-copilot`, `brooks-tester`, `brooks-language-lawyer`).

Direct dispatch by name is supported and is what the `assemble-with-grok-team` skill now uses (`subagent_type: "brooks-copilot"` etc., worktree isolation for Tester) — a pattern enabled by live smoke testing confirming the single-turn limit observed on named `subagent_type` spawns in Grok 0.2.43 is resolved as of Grok 1.0.3. See `.grok/agents/README.md` and the skill for details. The named brooks-* definitions provide the detailed contracts and remain useful for discovery (catalog / Ctrl+Shift+A) and manual one-off use.

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
  assemble-team/SKILL.md            Team briefing skill (all platforms)
  assemble-with-fleet/SKILL.md      Parallel team spawn via Copilot CLI fleet mode or OpenCode task tool
  assemble-with-grok-team/SKILL.md  Parallel spawn via Grok native subagents + worktrees + shared todo_write

.grok/                          Grok Build specific
  agents/                       Native agent defs (brooks-copilot, brooks-tester, brooks-language-lawyer)

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
| `.grok/agents/` | Grok Build | Native agent definitions (`brooks-copilot`, `brooks-tester`, `brooks-language-lawyer`) |
| `agents/` | Claude Code | Dispatch templates for subagent roles (Copilot, Tester, Language Lawyer) |
| `commands/` | Claude Code | Slash commands (`/assemble-team`, `/assemble-with-agent-teams`) |
| `.github/agents/` | Copilot CLI | Custom agent definitions for `/agent` dispatch |
| `.opencode/agents/` | OpenCode | Custom agent definitions for subagent dispatch |
| `skills/assemble-team/` | All platforms (Claude Code, OpenAI Codex, Grok Build, Copilot CLI & OpenCode) | Team briefing skill |
| `skills/assemble-with-grok-team/` | Grok Build | Parallel spawn via native `spawn_subagent`, worktrees, and shared `todo_write` |
| `skills/assemble-with-fleet/` | Copilot CLI & OpenCode | Parallel spawn via fleet mode or task tool |

> **Note:** Copilot CLI recognizes `.claude-plugin/` in addition to `.github/plugin/` when loading plugin manifests.

### OpenCode compatibility

The eight role skills in `skills/` are compatible with [OpenCode](https://opencode.ai) via the [Agent Skills open standard](https://github.com/agentskills/agentskills) — OpenCode reads `SKILL.md` files the same way Claude Code and Copilot CLI do. See [OpenCode skills documentation](https://opencode.ai/docs/skills/) for the full discovery path list and `SKILL.md` format reference.

Full agent dispatch is supported through `.opencode/agents/`, which includes Copilot, Tester, and Language Lawyer subagent definitions. These files use OpenCode's [agent frontmatter format](https://opencode.ai/docs/agents/) (`description`, `mode: subagent`, `permission` block). Permission keys map to specific tools: `edit` controls write/edit/patch operations, `bash` controls shell commands, `webfetch` controls web fetches, and `read` controls file reads (allowed by default). The Copilot sets `edit: deny`, `bash: deny`, and `webfetch: deny`; the Tester sets `edit: allow` and `bash: allow`. To restrict file reads: `read: deny`.

If OpenCode updates its agent frontmatter format, check the [OpenCode agent specification](https://opencode.ai/docs/agents/) to verify these files remain current.

> **Note for maintainers:** The core dispatch set (Copilot, Tester, Language Lawyer) is duplicated across five locations: `.codex/agents/` (Codex), `.grok/agents/` (Grok Build), `.opencode/agents/` (OpenCode), `agents/` (Claude Code), and `.github/agents/` (Copilot CLI). The four Codex-only specialists (Editor, Toolsmith, Program Clerk, Administrator) exist only in `.codex/agents/` and have no peer files on other platforms. This has already caused real drift more than once (a permission-grant fix landed with three different wordings; a skill exclusion was added to some copies and missed others, most recently when Grok's guard blocks were missing the same exclusion that had already been backported to the other four). When changing a role's dispatch template, update every platform copy that role has and check that these stay equivalent:
> - The protocol body (review steps, failure-mode checklist, output format)
> - The skill-exclusion list (`SUBAGENT-STOP` / `[CODEX-STOP]` / Grok's inline guard)
> - The tool/permission grant, expressed in each platform's own format (Claude Code `tools`/`disallowedTools`, OpenCode `permission`, Copilot CLI `tools`, Codex `sandbox_mode`, Grok `tools:`)
>
> With a fifth platform now in place, a single canonical source per role with generated or symlinked platform shims is worth serious consideration instead of continuing to hand-sync five copies.

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
