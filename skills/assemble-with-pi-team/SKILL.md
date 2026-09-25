---
name: assemble-with-pi-team
description: "Pi Coding Agent — Spawn a Brooks Surgical Team using the optional `subagent` example extension's OS-process parallel dispatch. Only applies if that extension is installed; pi-coding-agent core has no built-in subagent tool or task list. Use when running under Pi Coding Agent and the `subagent` tool is available; otherwise use `assemble-team` for single-session work."
---

# Assemble Pi Coding Agent Surgical Team (Parallel, Extension-Gated)

<SUBAGENT-STOP>
If you were spawned by the `subagent` tool, you are already playing one of these roles — your role contract is your `.pi/agents/*.md` (or `~/.pi/agent/agents/*.md`) frontmatter body. Do not invoke `assemble-with-pi-team`, `assemble-with-fleet`, `assemble-with-grok-team`, `assemble-with-hermes-team`, `using-brooks-team`, or `surgeon` — spawning further teammates from inside a subagent creates uncontrolled OS-process recursion.
</SUBAGENT-STOP>

## Check This First: Is the `subagent` Tool Even Available?

pi-coding-agent core ships no subagent/delegate tool and no built-in task list — this is confirmed by inspecting the full `ExtensionAPI` surface in `packages/coding-agent/src/core/extensions/types.ts`, which has no conversation-spawning primitive at all. The **only** way to get real parallel teammates on Pi is the official `examples/extensions/subagent/` extension, which is opt-in example code, not a first-party feature, and pre-1.0 (breaking changes have landed release-to-release, verified against v0.87.1).

Check your tool list for a `subagent` tool before doing anything else in this skill:
- **Present** → continue below.
- **Absent** → stop here and use the `assemble-team` skill instead for single-session work. Pi's AGENTS.md/`CLAUDE.md` context-file support works with zero setup. Agent Skills discovery needs no new adapter files either — a project-local `.agents/skills/` is gated by the same one-time project-trust decision as `.pi/skills/`, but symlinking this repo's `skills/` into the *global* `~/.agents/skills/` once (see [README's Pi Coding Agent section](../../README.md#pi-coding-agent)) or using `pi --skill <path>` for a one-off session both skip that trust step entirely — confirmed at the source level, not just documented behavior. Either way, `assemble-team` and every role skill are usable immediately even without the extension.

## Step 1: Quick Project Survey

Read just enough to know what work is ahead:
- Project name and primary language/framework
- Any open tasks, in-progress work, or recent git activity (`git log --oneline -5`)
- Presence of `AGENTS.md`, `docs/`, `tests/` directories

Do NOT do a full orientation — that is `assemble-team`'s job.

## Step 2: Decide the Team Composition

Ask the user one focused question:

> I'll spawn the core surgical team now (Surgeon = you + **Copilot** + **Tester**), via Pi's `subagent` extension.
>
> Do you want the **Language Lawyer** added too, for tricky framework/version edge cases?
>
> Reply with "yes"/"add language lawyer", or "default" to proceed with Copilot + Tester only.

Wait for the user's response before continuing. (Editor/Toolsmith/Program Clerk aren't offered here — this repo has no `.pi/agents/*.md` definition for them, so there's no `agentName` to dispatch in Step 4. They're still available inline via their own skills in the main session.)

## Step 3: No Shared Task List — Use Plain Notes

The `subagent` extension has no task-board equivalent; each spawned subagent is a separate OS process with its own isolated context and no visibility into anything but what you put in its prompt. Keep your own running plan (working notes, or Pi's `todo` example extension if the user has it installed) and paste the relevant slice into each subagent's task string — the plan lives in what you tell each child, same as this repo's Hermes `delegate_task` default mode.

## Step 4: Spawn the Team

Call the `subagent` tool with `mode: "parallel"` and one `tasks[]` entry per role (max 8 tasks / 4 concurrent — hardcoded in the extension, not configurable). For each task:

- `agentName`: `"copilot"`, `"tester"`, or `"language-lawyer"` — these resolve to `.pi/agents/*.md` in this repo, **only if the call also sets `agentScope: "project"` or `"both"`** (default is `"user"`-scope only, which silently ignores `.pi/agents/`). If the user has instead symlinked these role files into the global `~/.pi/agent/agents/` (see the [README's Pi Coding Agent section](../../README.md#pi-coding-agent)), omit `agentScope` entirely — `"user"` is the default and resolves from there in any project.
- `task`: the role's full assignment in one shot — project summary, the concrete diff/spec/question, file ownership (below). There is no lightweight-first-turn/resume split; each subagent is a fresh process that only knows what's in this string.

```json
{
  "mode": "parallel",
  "agentScope": "project",
  "tasks": [
    {
      "agentName": "copilot",
      "task": "You are the Copilot on the surgical team for [PROJECT]. [project summary]. Review [WHAT_WAS_IMPLEMENTED] against [SPEC_OR_PLAN]. Diff/files: [paste — this role has no bash, so it cannot run git itself]. Report in the Blocking/Important/Suggestions format from your role contract."
    },
    {
      "agentName": "tester",
      "task": "You are the Tester on the surgical team for [PROJECT]. [project summary]. Enumerate failure modes for [WHAT_IS_BEING_TESTED] per your role contract, then write the tests. Do not touch production code."
    }
  ]
}
```

Add a third `language-lawyer` task if selected in Step 2, with `[THE_EXACT_QUESTION]`, `[RUNTIME_CONTEXT]`, and note it has no dedicated web-fetch/web-search tool (see `.pi/agents/language-lawyer.md`) — it can only verify via `bash` and whatever's reachable that way.

Since each task runs in an **isolated OS process with its own working directory context** (not a shared in-process session), there is no live conflict detection between concurrent writers the way a shared editor session would have. Only Tester writes files here — never run two write-capable tasks with overlapping file ownership in the same parallel batch.

### File Ownership (Include in Every Task String)

- **Copilot**: read-only (`tools: read, grep, find, ls` in its agent file — a real enforced boundary, not a convention). Never edits source.
- **Tester**: owns `tests/`, `**/*.test.*`, `**/*.spec.*`. Full tool access (its agent file omits `tools:` to inherit the default set).
- **Language Lawyer**: read-only research + `bash` for verification experiments only (`tools: read, grep, find, ls, bash`). Never edits files.

## Step 5: Handoff

The `subagent` tool streams each task's progress and returns when all complete (or per-task, depending on how your Pi session surfaces it) — there's no background-and-poll model to manage here, unlike Claude Agent Teams or Hermes' kanban dispatcher. Tell the user:

> Team spawned via Pi's `subagent` extension (parallel mode, `agentScope: "project"`).
>
> - **Copilot** and **Language Lawyer** (if added) are read-only — their findings come back as their task's output, nothing to apply.
> - **Tester** writes directly to this working tree (no worktree isolation — the extension spawns separate OS processes, not separate checkouts). Review its changes with `git diff` before continuing.
> - There's no shared task list on this platform — track completion from each task's returned output, same as Hermes' default `delegate_task` mode.
> - Run `assemble-team` any time for the full role reference.

## Anti-Patterns to Prevent

- Do not assume the `subagent` tool exists — check first (see top of this skill). Bare pi-coding-agent has no parallel-agent mechanism at all.
- Do not pass `agentScope` as `"user"` (the default) and then expect `.pi/agents/*.md` to be found — it won't be.
- Do not run two write-capable tasks (e.g. two Tester-like tasks) with overlapping file ownership in the same `parallel` batch — there is no worktree or checkout isolation between them.
- Do not treat a subagent's self-reported result as verified — re-run tests yourself before telling the user the work is done.
- Do not spawn the full team for tiny changes.
