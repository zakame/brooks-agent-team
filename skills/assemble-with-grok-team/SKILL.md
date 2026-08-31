---
name: assemble-with-grok-team
description: "Grok-native surgical team — Spawn a Brooks Surgical Team using parallel subagents. One independent agent per role with shared todo list, worktree isolation for writers, and native background task monitoring. Use when you want true parallel teammates (Copilot reviews while you continue implementing)."
---

# Assemble Grok Surgical Team (Parallel)

<SUBAGENT-STOP>
You are already operating as a dispatched subagent (Copilot, Tester, Language Lawyer, etc.). Do not invoke `assemble-with-grok-team`, `assemble-with-fleet`, `using-brooks-team`, or `surgeon` skills. Spawning further teammates from inside a subagent creates uncontrolled recursion.
</SUBAGENT-STOP>

When invoked, execute the following steps in order.

## Step 1: Quick Project Survey

Read just enough to understand the work ahead:

- Project name / primary language / framework (from README, package.json, Cargo.toml, pyproject.toml, etc.)
- Recent activity: `git log --oneline -5` and current branch
- Presence of `tests/`, `docs/`, `AGENTS.md`, `CLAUDE.md`, or similar
- Any obvious large modules or risky areas from the top-level structure

Do **not** do a full orientation. This skill is about getting the team spawned quickly.

Record a one-paragraph summary for use in subagent prompts.

## Step 2: Decide the Team Composition

Ask the user one focused question:

> I'll spawn the core surgical team now (Surgeon = you + **Copilot** + **Tester**).
>
> Do you want any additional teammates?
>
> - **Language Lawyer** — for tricky language, framework, or version edge cases
> - **Editor** — for documentation-heavy work
> - **Toolsmith** — if you need new automation or scripts
> - **Program Clerk** — if the codebase needs reorganization alongside the work
>
> Reply with the roles to add, or say **"default"** (or just "go") to proceed with Copilot + Tester only.

Wait for the user's explicit response before continuing.

Core team (always spawned):
- Copilot (read-only review agent)
- Tester (adversarial test agent, runs in isolated worktree)

## Step 3: Initialize the Shared Task List

Use the `todo_write` tool to create the initial shared task list.

Use these task tags so roles know what to claim:
- `[implement]` — Surgeon claims these (you)
- `[review]` — Copilot claims these (depend on corresponding `[implement]`)
- `[test]` — Tester claims these (depend on corresponding `[implement]`)
- `[research]` — Language Lawyer claims these (if spawned)
- `[docs]` — Editor claims these (if spawned)
- `[tooling]` — Toolsmith claims these (if spawned)
- `[structure]` — Program Clerk claims these (if spawned)

For every significant piece of work identified in the survey, create at least:
- One `[implement]` task
- One dependent `[review]` task
- One dependent `[test]` task

Add clear acceptance criteria on the implement tasks. Link dependencies explicitly in the task descriptions.

Example structure (adapt to actual project):

```
- [implement] Core feature X (acceptance: ... )  owner: Surgeon
  - [review] Review feature X                        owner: Copilot   (blocked until parent done)
  - [test]  Test feature X                           owner: Tester    (blocked until parent done)
```

Call `todo_write` now with the initial list. This becomes the single source of truth for the whole team.

## Step 4: Spawn the Team (Parallel)

Spawn all selected teammates in a **single response** using parallel `spawn_subagent` tool calls — each with the teammate's full task in one shot. There is no lightweight-first-turn/resume-turn split: give each teammate everything it needs up front (project summary, todos, file ownership, the actual task) and let it run to completion in the background.

### Why direct dispatch (confirmed 2026-08-14)
Earlier revisions of this skill spawned every role via the built-in `general-purpose` agent with a deliberately lightweight first-turn prompt, then used `resume_from` to deliver the real task — a workaround for an observed limitation on Grok 0.2.43 where named `.grok/agents/*.md` agents got only a single turn on direct `subagent_type` dispatch. Live headless smoke testing (`grok -p` + `--output-format streaming-messages-json`, 4 runs) confirmed this limitation is resolved on Grok 1.0.3: `brooks-copilot`, `brooks-tester` (with `isolation: "worktree"`), and `brooks-language-lawyer` each completed real multi-tool-call tasks (2-4 tool calls, scaling with task size) fully autonomously in the background on direct dispatch — `resume_from` was never called. This skill now spawns named agents directly with their full task; `resume_from` is kept only as an optional follow-up mechanism (see Monitoring below).

### The real `spawn_subagent` parameters
Per Grok's own subagent documentation:

| Parameter | Description |
|---|---|
| `prompt` | The full task prompt for the subagent. |
| `description` | Short label (3-5 words). |
| `subagent_type` | Agent type to launch — a `brooks-*` name for Copilot/Tester/Language Lawyer (they have `.grok/agents/*.md` definitions), `general-purpose` for Editor/Toolsmith/Program Clerk (no native Grok agent definition exists for those three). |
| `background` | Always `true` in this skill — run in the background, return a subagent ID immediately. |
| `isolation` | `worktree` for any role that edits files; omit (shared workspace) for read-only roles. |
| `resume_from` | Continues a *completed* subagent's conversation with full context inherited. Use it for a genuine follow-up round (e.g. asking Copilot a clarifying question after its review lands) — not to "unblock" a task, since there's nothing to unblock anymore. |

**`capability_mode` is not a `spawn_subagent` parameter (corrected 2026-09-01).** Earlier revisions of this skill passed `capability_mode` (`read-only`/`read-write`/`execute`/`all`) directly in every spawn call, based on documentation that turned out to be superseded. Current Grok subagent documentation states plainly that "capability mode is not a spawn argument," and lists `spawn_subagent`'s real parameters as exactly the six above. A role's default capability instead comes from its agent type and any `[subagents.roles.<name>].default_capability_mode` set in `.grok/config.toml`. This repo doesn't set that, since the confirmed working restriction mechanism is frontmatter `tools:` (see the Copilot/Language Lawyer notes below), not capability mode. Do not include `capability_mode` in spawn JSON; it does nothing there.

### Common Rules for All Spawns
- Set `background: true`.
- Give each teammate its full task in the initial `prompt` — project summary, the specific todo item(s) it owns, file ownership boundaries, and the concrete task (spec/diff/question to investigate). Don't hold anything back for a later resume.
- Include the current todo snapshot (or tell it to read the live list via `todo_write`).
- Tell it to claim its tagged tasks at the start of its run.
- Point it at its role contract (`.grok/agents/brooks-*.md`, kept in sync with the cross-platform skill of the same name) as a one-line reminder — it isn't required reading before acting, since the agent definition's body already governs the subagent's behavior, but it's a cheap, harmless pointer if the teammate wants to double check a protocol detail.

### File Ownership Rules (Include in Every Prompt)

- **Copilot**: Read-only across the entire repo. Never edits source. Can update todos.
- **Tester**: Owns `tests/`, `**/*.test.*`, `**/*.spec.*`, and test utilities. May run commands and edit only within that scope. Works in an isolated worktree.
- **Language Lawyer**: Read-only research. May use web search and fetch. Never edits files.
- **Editor**: Owns `docs/`, `README*`, `*.md` (except code-adjacent notes), and documentation sources.
- **Toolsmith**: Owns `scripts/`, tooling configs, Makefiles, and new automation. Every tool must have `--help` and tests.
- **Program Clerk**: Proposes structure changes only. Executes renames/refactors only after Surgeon approval via todo or explicit message.

  (Clarification for Editor: "code-adjacent notes" means scratch notes, TODO markers, or similar embedded alongside source code or in implementation directories. Pure project documentation (top-level and docs/ READMEs, CHANGELOG, specs, AGENTS.md-style files, and other standalone *.md) belongs to Editor. When in doubt on a *.md file, the Surgeon decides and records the assignment in the todo list.)

To support the checklist in practice, include a "Current writers" note in the initial `todo_write` snapshot and in every spawn prompt's "Current todos" section (e.g. "Writers active: none — Tester worktree pending apply"). This makes the live state visible to all teammates.

### Copilot Spawn (Always)

Spawn `brooks-copilot` directly with its full review task — no `general-purpose` indirection, no resume needed. Its role contract is `.grok/agents/brooks-copilot.md` (kept in sync with the cross-platform `copilot` skill); that definition's body governs its behavior once spawned. Its `tools:` frontmatter is a confirmed, real hard restriction (write and edit tools are genuinely unavailable to it) — the only Brooks role on Grok Build with an actually-enforced read-only boundary rather than a prompt-level convention.

```json
{
  "description": "Brooks Copilot — design-intent review for [project]",
  "subagent_type": "brooks-copilot",
  "background": true,
  "prompt": "You are the Copilot on the surgical team for [PROJECT].\n\n[project summary]\n\nYour role contract is `.grok/agents/brooks-copilot.md` (kept in sync with the cross-platform `copilot` skill).\n\nClaim your `[review]` tasks via `todo_write`, then review [WHAT_WAS_IMPLEMENTED] against [SPEC_OR_PLAN]. [Provide the diff, base SHA, or the specific files to inspect — in read-only mode it cannot run git itself.]\n\nFile ownership: strictly read-only.\n\nCurrent todos:\n[PASTE CURRENT TODO SNAPSHOT]\n\nRun the full review now and report your findings in the mandated Blocking/Important/Suggestions format."
}
```

### Tester Spawn (Always)

Spawn `brooks-tester` directly with `isolation: "worktree"` and its full test task. Role contract: `.grok/agents/brooks-tester.md` (kept in sync with the cross-platform `tester` skill).

```json
{
  "description": "Brooks Tester — adversarial test strategy and implementation",
  "subagent_type": "brooks-tester",
  "background": true,
  "isolation": "worktree",
  "prompt": "You are the Tester on the surgical team for [PROJECT].\n\n[project summary]\n\nYour role contract is `.grok/agents/brooks-tester.md` (kept in sync with the cross-platform `tester` skill) — follow the failure-mode enumeration before writing any test.\n\nClaim your `[test]` tasks via `todo_write`. You are in an isolated worktree — report its path in your result.\n\n[WHAT_IS_BEING_TESTED], [SPEC_OR_DESIGN], [KNOWN_RISKY_AREAS].\n\nFile ownership: strictly limited to tests/, *.test.*, *.spec.* and test utilities.\n\nCurrent todos:\n[PASTE CURRENT TODO SNAPSHOT]\n\nRun the full test strategy and implementation now."
}
```

Note: `brooks-tester.md` carries no `tools:` restriction, since the Tester needs full read/write/execute access to write and run test files. This is intentional, matching how the Tester subagent is left unrestricted on every other platform. The `tests/`-only restriction is enforced by convention and Surgeon review, not by any platform mechanism (Grok has no path-scoped tool restriction).

### Optional Role Spawns

**Language Lawyer** (only if requested) — spawn `brooks-language-lawyer` directly, role contract `.grok/agents/brooks-language-lawyer.md` (kept in sync with the cross-platform `language-lawyer` skill):

```json
{
  "description": "Brooks Language Lawyer — research on [TOPIC]",
  "subagent_type": "brooks-language-lawyer",
  "background": true,
  "prompt": "You are the Language Lawyer on the surgical team for [PROJECT].\n\n[project summary]\n\nYour role contract is `.grok/agents/brooks-language-lawyer.md` (kept in sync with the cross-platform `language-lawyer` skill) — cite sources, never guess.\n\nClaim your `[research]` tasks via `todo_write`.\n\n[THE_EXACT_QUESTION], [RELEVANT_CODE], [RUNTIME_CONTEXT].\n\nCurrent todos:\n[PASTE CURRENT TODO SNAPSHOT]\n\nRun the full Research Protocol now and report in the mandated Language Lawyer Finding format."
}
```

Note: `.grok/agents/brooks-language-lawyer.md`'s `tools:` frontmatter grants `read_file, list_dir, search_tool, use_tool, web_search, run_terminal_command`, matching this role's cross-platform grant (WebFetch/WebSearch/Bash, no edit) and its own contract's allowance for small runtime experiments via Bash. Unlike Copilot, this **does not fully close the write gap for this role**: live testing found that including `run_terminal_command` (needed for its Bash-experiment allowance) also unlocks the `search_replace` edit tool as a side effect. This role's "never edit" boundary therefore still relies on its own prompt contract, not a platform restriction (see the `.grok/agents/brooks-language-lawyer.md` note for the full finding).

**Editor / Toolsmith / Program Clerk** (if requested) — these three have **no** `.grok/agents/*.md` definition, so they still spawn via `general-purpose`, with the prompt starting with an explicit skill-invocation instruction (there's no agent-definition body to fall back on for the role contract). They get their full task in the same call — no lightweight-then-resume split, same as the named roles above:
- `subagent_type`: `"general-purpose"`
- `isolation`: `"worktree"`

#### Full spawn examples for optional writing roles (adapt [PROJECT], summary, and todos)

**Editor (documentation work):**
```json
{
  "description": "Brooks Editor — documentation accuracy and completeness for [project]",
  "subagent_type": "general-purpose",
  "background": true,
  "isolation": "worktree",
  "prompt": "At the start of this session, invoke the `editor` skill — that is your role contract.\n\nYou are the Editor on the surgical team for [PROJECT].\n\n[project summary]\n\nClaim your `[docs]` tasks via `todo_write`. You are in an isolated worktree — report its path in your result.\n\n[WHAT_DOCS_NEED_WORK], [ACCEPTANCE_CRITERIA].\n\nFile ownership: owns `docs/`, `README*`, `*.md` (except code-adjacent notes), and documentation sources.\n\nCurrent todos:\n[PASTE CURRENT TODO SNAPSHOT]\n\nRun the full documentation review and verification now."
}
```

**Toolsmith (automation work):**
```json
{
  "description": "Brooks Toolsmith — custom automation for [project]",
  "subagent_type": "general-purpose",
  "background": true,
  "isolation": "worktree",
  "prompt": "At the start of this session, invoke the `toolsmith` skill — that is your role contract.\n\nYou are the Toolsmith on the surgical team for [PROJECT].\n\n[project summary]\n\nClaim your `[tooling]` tasks via `todo_write`. You are in an isolated worktree — report its path in your result.\n\n[WHAT_NEEDS_AUTOMATING].\n\nFile ownership: owns `scripts/`, tooling configs, Makefiles, and new automation. Every tool must have `--help` and tests.\n\nCurrent todos:\n[PASTE CURRENT TODO SNAPSHOT]\n\nBuild and verify the tool now."
}
```

**Program Clerk (structure work):**
```json
{
  "description": "Brooks Program Clerk — code organization for [project]",
  "subagent_type": "general-purpose",
  "background": true,
  "isolation": "worktree",
  "prompt": "At the start of this session, invoke the `program-clerk` skill — that is your role contract.\n\nYou are the Program Clerk on the surgical team for [PROJECT].\n\n[project summary]\n\nClaim your `[structure]` tasks via `todo_write`. You are in an isolated worktree — report its path in your result.\n\n[WHAT_NEEDS_REORGANIZING] — the Surgeon has already approved this refactor via [todo item / explicit message].\n\nFile ownership: proposes structure changes; executes renames/refactors only after Surgeon approval (already given for this dispatch).\n\nCurrent todos:\n[PASTE CURRENT TODO SNAPSHOT]\n\nRun the reorganization now."
}
```

### Monitoring and Optional Follow-Up

- Use `get_command_or_subagent_output(task_ids=["<SUBAGENT-ID>"], timeout_ms=<N>)` to retrieve a background teammate's result once it completes — the result includes `tool_calls`/`turns` metadata plus the final text.
- `resume_from` is available for a genuine follow-up round after a subagent has **completed** (e.g. "Copilot, elaborate on finding #2") — pass its subagent ID and a fresh prompt with any new context (the resumed child re-renders its system prompt/tools from the current agent definition, but doesn't automatically see anything you haven't told it). It is not needed to get a subagent's first task done.
- Stand down an idle teammate with `kill_command_or_subagent(task_id)`.

**Worktree path discovery, if `x.ai/git/worktree/*` extension methods aren't in your tool list** (they're capability-gated): every writing teammate's spawn prompt asks it to report its worktree path in its result, or list worktrees directly with `grok worktree list` (a regular shell command, always available as of Grok 1.0.3 — see "Reviewing and Applying Worktree Changes" below for the full review/apply workflow).

Spawn all agents together. Do not wait for responses between spawns.

## Step 5: Handoff and Monitoring Instructions

After the parallel spawn calls complete, tell the user:

```
Team is live.

You are the Surgeon. Claim `[implement]` tasks from the shared todo list (Ctrl+T to open the todos pane) and continue the critical path.

Your teammates are running in the background, spawned directly by name with their full task:
- Copilot (`brooks-copilot`, read-only review)
- Tester (`brooks-tester`, in isolated worktree)
- [any others]

Monitoring:
- Press **Ctrl+T** to see the shared todo list (todo_write state).
- Press **Ctrl+G** to see the tasks pane (active subagents + background tasks).
- Use the queue pane (Ctrl+;) for prompt queue / background task status.
- To check a specific teammate: use get_command_or_subagent_output with their task ID (shown in the UI).
- To stand down an idle teammate: kill_command_or_subagent(task_id).

Worktree Safety (critical for Tester and any editing roles):
- Tester (and Editor/Toolsmith when spawned with `isolation: "worktree"`) run in fully isolated git worktrees.
- **Never apply blindly.** Changes are invisible to the main workspace until you explicitly merge them.
- See the full "Reviewing and Applying Worktree Changes (Tester / Editor)" section below for the concrete command set (`x.ai/git/worktree/list`, `apply`, `status`, `remove`, `gc`), review workflow, examples (with TUI caveats), conflict handling, and non-negotiable Surgical Team safety rules.
- Copilot and Language Lawyer are strictly read-only (no worktree, no edits possible).
- Ownership rules remain absolute — never allow overlapping file writes across teammates.

Next actions:
- Start claiming your first `[implement]` task.
- When a unit is ready for review, the Copilot will pick up the dependent `[review]` task.
- Run `assemble-team` at any time for the full role reference (including optional Editor/Toolsmith for docs work).
- **Before spawning or resuming any writing role (Tester/Editor/Toolsmith/Program Clerk), run the Surgeon checklist in the Safety rules section below.** Never allow two writers with potentially overlapping ownership at the same time — serialize (Tester first for the safety net) and document current writers in the todo snapshot you pass to spawns.
- When the session is done, cancel any remaining background teammates.
```

## Reviewing and Applying Worktree Changes (Tester / Editor)

Writing teammates run with `isolation: "worktree"`. Their filesystem changes live in a completely separate working tree under `~/.grok/worktrees/...` and have **zero effect** on your main workspace until you merge them.

### Why this matters for the Surgical Team
- Tester can freely write and run tests in isolation without polluting your in-progress implementation.
- Editor can polish docs/READMEs without fighting your concurrent edits.
- You (the Surgeon) retain final control and visibility — exactly as Brooks intended (the Surgeon never loses the thread).

### Discovery
- The worktree path is **required** to be surfaced by the teammate: the spawn prompts for writing roles (Tester, and Editor/Toolsmith when using `isolation: "worktree"`) explicitly instruct them to report it in their final result.
- Active background worktrees also appear in the queue pane (Ctrl+;) and tasks pane (Ctrl+G). The shared todo list is in the todos pane (Ctrl+T).
- You can list them via platform tools: `x.ai/git/worktree/list` (when available to your agent session) or by inspecting `~/.grok/worktrees/<project-hash>/` on disk.

**Example filesystem layout** (actual `x.ai/git/worktree/list` output may be structured differently — use the tool or path inspection for precise IDs):
```
~/.grok/worktrees/<project>/
  subagent-<SUBAGENT-ID-1>/
  subagent-<SUBAGENT-ID-2>/
  subagent-<SUBAGENT-ID-3>/
```

### Review commands (run from your main session)
Platform extension methods (appear in your tool list with the right capability):

- `x.ai/git/worktree/status` — progress / health of worktree sessions
- `x.ai/git/worktree/list` — enumerate active worktrees for the current project (preferred for obtaining usable IDs/paths)

Standard git inspection (run from your main workspace, targeting the worktree path):

- `git -C <full-worktree-path> status --short` and `git -C <full-worktree-path> diff HEAD`
- For untracked files added by the teammate: `git -C <path> ls-files --others --exclude-standard`

### Apply (merge) workflow — only after review
Once you are satisfied with the teammate's output (after reading the diff):

```
# Example: apply Tester's test changes after obtaining the ID via list or first-turn confirmation
x.ai/git/worktree/apply subagent-<SUBAGENT-ID>
# or using full path:
# x.ai/git/worktree/apply ~/.grok/worktrees/<project>/subagent-<SUBAGENT-ID>/
```

**Important TUI caveats:**
- `x.ai/git/worktree/apply` (and the other `x.ai/git/worktree/*` commands) are **Grok TUI extension methods**, not regular shell commands. They only appear in your available tools when your current session has the appropriate capability.
- The platform will confirm the merge result (or surface conflicts). Always run `git status` and inspect the result immediately after applying.
- If the extension command is not available in your tool list, fall back to manual inspection + `git cherry-pick` or file-by-file copy from the worktree path as a last resort (rare).

The platform merges the isolated changes into your current working directory. It respects concurrent edits where possible; on conflicts you will typically see platform feedback or a prompt — review the result before continuing.

### Handling Conflicts and Concurrent Edits
When the main workspace and a writing teammate's worktree have touched the same files:

- **Recommended:** Commit or stash your local changes in the main workspace before applying any worktree. This gives the cleanest merge.
- After `x.ai/git/worktree/apply`, immediately run `git status` and `git diff --stat` (or open the diff in your editor) to see what actually landed.
- If the platform reports conflicts, resolve them the normal git way (`git mergetool`, manual edit, `git add`, `git commit`).
- For very small overlapping changes, you can also manually copy specific files from the worktree path into your main tree and commit — this is sometimes clearer than a conflicted apply.
- **Never** have two writing teammates (Tester + Editor, Tester + Toolsmith, etc.) whose ownership areas could overlap running in worktrees at the same time. See the Safety rules below for the exact guardrail.

### Cleanup
After a successful apply (or when discarding):

- `x.ai/git/worktree/remove <id>` — tear down the worktree (TUI extension method; same caveats as apply)
- `x.ai/git/worktree/gc` — garbage-collect stale worktrees

Example:
```
x.ai/git/worktree/remove subagent-<SUBAGENT-ID>
```

### Safety rules (non-negotiable)
- Always read the diff before applying. A malicious or confused Tester could in theory introduce bad tests.
- **Never have two writing teammates in worktrees at the same time if their ownership areas could overlap.** This is the most common source of painful conflicts.
  - **Surgeon checklist before spawning any writing role:**
    1. Is there already a writing teammate (Tester/Editor/Toolsmith) with an active worktree? If yes → wait for it to finish, review + apply + remove its worktree, *then* spawn the next writer.
    2. If you truly need parallel writing (rare), ensure **completely disjoint scopes** (e.g. Tester strictly limited to `tests/`, Editor strictly to `docs/` and top-level markdown) and document the separation in the spawn prompts + todo items.
    3. When in doubt, serialize: Tester first (tests are the safety net), then Editor.
- If a worktree contains changes you do **not** want, simply kill the subagent and run `x.ai/git/worktree/remove`. The main workspace is untouched.
- Worktree isolation is a **feature**, not a limitation. It is the mechanism that lets the Surgical Team achieve true parallel editing safely while the Surgeon retains final control and visibility.

This section exists because the underlying Grok platform documentation for `x.ai/git/worktree/*` is still light (as of 2026-08). The Surgical Team pattern makes these commands first-class concerns. Note: Grok 1.0.3 also ships a first-class top-level `grok worktree list/show/rm/gc` CLI subcommand (plus `--worktree`/`--worktree-ref`/`--restore-code` flags on the main `grok` command) as an always-available alternative to the TUI-extension methods below — consider it for a future simplification pass, since it doesn't depend on capability-gating.

## Important Grok-Specific Notes (Internal)

- **Direct dispatch (confirmed 2026-08-14):** Custom agents from `.grok/agents/*.md` (brooks-*) were observed to receive only 1 turn when used as `subagent_type` on Grok 0.2.43 — that limitation is resolved on Grok 1.0.3 (live headless smoke testing: all three roles completed real multi-tool-call tasks fully autonomously on direct dispatch, no `resume_from` needed), so this skill now spawns `brooks-copilot`/`brooks-tester`/`brooks-language-lawyer` directly by name with their full task in one call. Editor/Toolsmith/Program Clerk have no `.grok/agents/*.md` definition and still spawn via `general-purpose`, invoking their cross-platform skill explicitly in the prompt — but they too get their full task up front now, not a lightweight-then-resume split.
- The `brooks-copilot`, `brooks-tester`, and `brooks-language-lawyer` definitions in `.grok/agents/` are both the spawn target (via `subagent_type`) and discoverable for manual dispatch (`grok inspect`, Ctrl+Shift+A catalog). They carry the full Surgical Team protocols in their body text.
- For plugin installs, users may need to ensure `.grok/agents/` from the plugin is linked or copied.
- `todo_write` is the single source of truth for task state across the whole team.
- Worktree changes from Tester (or other writing agents) are isolated until the Surgeon explicitly applies them. See the "Reviewing and Applying Worktree Changes (Tester / Editor)" section above for the full command set, concrete examples, conflict handling, and non-negotiable safety rules (including the Surgeon checklist for writers). This is a safety feature, not a bug.
- `resume_from` is kept only for genuine follow-up rounds after a subagent completes — not part of getting a teammate's first task done.

## Anti-Patterns to Prevent

- Do not spawn the full team for tiny changes.
- Do not let the Surgeon context-switch instead of using the Administrator role when priorities shift.
- Do not have teammates edit overlapping files.
- Do not mark tasks complete without acceptance criteria being met and verified by the owning role.

This is the Grok-native realization of the Surgical Team. One Surgeon, many focused supporters, real parallelism, and protected focus.