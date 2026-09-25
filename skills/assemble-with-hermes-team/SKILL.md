---
name: assemble-with-hermes-team
description: "Hermes Agent — Spawn a Brooks Surgical Team using Hermes's delegate_task tool for concurrent role dispatch, tracked on Hermes's native kanban board when your profile has the kanban toolset enabled, via the `hermes kanban` CLI dispatcher if requested and the toolset isn't there, or a plain/todo_list plan as a last resort. Use when running under the Hermes Agent harness and you want true parallel teammates."
---

# Assemble Hermes Surgical Team (Parallel)

<SUBAGENT-STOP>
If you are already a `delegate_task` child, or a kanban-dispatched worker (`HERMES_KANBAN_TASK` set in your environment), you are already playing one of these roles. Do not invoke `assemble-with-hermes-team`, `assemble-with-fleet`, `assemble-with-grok-team`, `assemble-with-pi-team`, `using-brooks-team`, or `surgeon`. A `delegate_task` child cannot itself call `delegate_task` as long as it's spawned with the default `role` (omitted → `"leaf"`) and your profile's `delegation.max_spawn_depth` stays at its default of `1` — Hermes' own docs describe `role` (`"leaf"`/`"orchestrator"`) and `max_spawn_depth` as exactly what governs recursion, not something separate from it. Don't pass `role: "orchestrator"` here, and don't assume recursion is blocked if something else has raised `max_spawn_depth`. (Whether a given deployed Hermes version's live tool schema actually surfaces the `role` field to the model varies — one field-tested session saw only `goal`/`context`/`output_schema` on the live tool; check your version before depending on `role` being present at all.)
</SUBAGENT-STOP>

## Which Mode: `delegate_task` or the Kanban Dispatcher?

Two ways to run a Hermes team — pick based on the work, not habit:

- **`delegate_task` (default)** — right when you're one Surgeon session shipping a single bounded artifact in this chat. Ephemeral, no board setup, fastest to start. Nothing survives the session ending except what you wrote down.
- **Kanban dispatcher** — right when the plan needs to outlive this chat, or fan out across dedicated Hermes profiles. Needs either the `kanban` toolset on your profile, or the `hermes kanban` CLI (works even without the toolset — see Step 3), plus something actually advancing cards (the gateway, or you looping `hermes kanban dispatch`).

Default to `delegate_task` unless the user asked for the dispatcher, wants the plan to persist past this session, or dedicated per-role profiles already exist.

When invoked, execute the following steps in order.

## Step 0: Prerequisites

- **Skill discovery.** This repo's skills live in `skills/` (mirrored at `.agents/skills/`), a project-local directory Hermes recognizes per the Agent Skills standard. Project-local skill directories require a one-time trust step before Hermes loads them:
  ```
  hermes skills trust <path-to-this-repo>
  ```
  Run once per clone; skip if already trusted. Run it against the repo you're implementing in, not the skills repo itself if they differ. For a permanent setup that needs no trust step and works from any project, add this repo's `skills/` path to `skills.external_dirs` in `~/.hermes/config.yaml` instead (supports `~`/`${VAR}` expansion) — see the [README's Hermes Agent section](../../README.md#hermes-agent).
- **Kanban toolset (check before relying on Step 3, Path A).** A regular `hermes chat` session has **zero** `kanban_*` tools by default — per Hermes' own docs, they only appear if "the active profile explicitly enables the `kanban` toolset for orchestrator work." Check now whether `kanban_create`/`kanban_list`/`kanban_link`/`kanban_unblock` are actually in your tool list.
- **CLI kanban, independent of the toolset above.** Even with no `kanban_*` tools, the `hermes kanban` shell CLI may still work (`hermes kanban create`, `list`, `dispatch`, `assignees`, etc. — check `hermes kanban --help` for your version). If the user asked for the dispatcher, use the CLI path in Step 3 rather than dropping straight to notes.
- **Kanban dispatcher.** Cards only advance on their own if something is actually driving them — on by default via `kanban.dispatch_in_gateway: true` in `~/.hermes/config.yaml` (siblings: `dispatch_interval_seconds: 60`, `review_dispatch: true`). If that's `true` but no gateway process is actually running, or it's `false`, cards will sit in `ready` forever: tell the user, then either help start the gateway or loop `hermes kanban dispatch` yourself until the board drains.

## Step 1: Quick Project Survey

Read just enough to understand the work ahead:

- Project name / primary language / framework (README, package.json, Cargo.toml, pyproject.toml, etc.)
- Recent activity: `git log --oneline -5` and current branch
- Presence of `tests/`, `docs/`, `AGENTS.md`, or similar
- Any obvious large modules or risky areas from the top-level structure
- Whether there's anything to review yet: an empty or near-empty tree means there's no diff for Copilot and no code-vs-spec for Tester to check — see the greenfield rule in Step 4.

Do **not** do a full orientation — this skill is about getting the team spawned quickly. Record a one-paragraph summary for use in `delegate_task` calls.

## Step 2: Decide the Team Composition

Skip this question if the user's invocation already named the team or mode (said "default", "go", "dispatcher", or gave a single bounded spec with no team discussion) — proceed straight to Step 3 with the core team. You can still offer additional roles later if new work surfaces (a research question, docs, tooling); don't gate on it up front.

Otherwise, ask the user one focused question:

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

Core team (always spawned): Copilot (read-only review), Tester (adversarial tests).

## Step 3: Initialize the Shared Task Board

Hermes has no in-memory shared-list tool like Claude Code's `TaskCreate`/`TaskList`/`TaskUpdate`/`TaskGet` or Grok's `todo_write`. Depending on what's actually available in this session, use one of three paths:

**Path A — kanban toolset enabled** (`kanban_create`/`kanban_list`/`kanban_link`/`kanban_unblock` are in your tool list):

- `kanban_create(title, assignee)` — one card per unit of work. Set `assignee` to the role name (`copilot`, `tester`, `language-lawyer`, `editor`, ...).
- `kanban_link(parent_id, child_id)` — add a dependency edge after creation, e.g. blocking a `[review]` card on its `[implement]` card.
- `kanban_list(...)` — check board state (supports filters, exact set undocumented — inspect what your Hermes version accepts).
- `kanban_unblock(task_id)` — release a card once its blocker resolves.

Card states flow `triage → todo → ready → running → blocked → review → done → archived`; a `kanban_link` edge auto-promotes a card from `todo` to `ready` once every parent card reaches `done`. Create the `[implement]` card, then its dependent `[review]`/`[test]` cards, then `kanban_link` each to its parent, before spawning anyone — the auto-promotion depends on that order.

**Path B — no kanban toolset, but the user asked for the dispatcher (or per-role profiles already exist):**

Field-verified from a live Hermes TUI session — use the `hermes kanban` CLI directly:

- Discover real assignees first: `hermes profile list` and/or `hermes kanban assignees`. **Never assign a card to a role name that isn't an actual profile** (`copilot`, `tester` are not profiles unless someone created them) — an unknown assignee silently never spawns a worker. If no per-role profiles exist, assign everything to `default` and paste the role's `SKILL.md` body into the card so the worker knows which contract to play (`--skill copilot`, `--skill tester`, etc., as your version supports).
- Set `--parent` at creation time, not a separate link step afterward — e.g. `hermes kanban create --parent <parent-id> --assignee <profile> --workspace dir:<path> --json ...`. A child created and only linked later can briefly read as unblocked and race its still-running parent. Use an idempotency key if your version supports one, so a retried create doesn't duplicate the graph.
- **Same-file cards can't fan out.** If two `[implement]` cards would touch the same file, chain them with `--parent` instead of running them concurrently — two workers on one file will race. Copilot/Tester cards should depend on the *last* card in that chain, not each `[implement]` card individually.
- **Board pin.** A Hermes TUI session pins `HERMES_KANBAN_BOARD` at chat boot. `hermes kanban boards create --switch` (or `boards switch`) only updates `~/.hermes/kanban/current`, which loses to that pinned env var — a newly created board can sit empty while cards keep landing on `default`. Export it explicitly and pass it on every mutating call: `export HERMES_KANBAN_BOARD=<slug>` **and** `--board <slug>`.
- See the dispatcher-gate note in Step 0 — check it before assuming cards will move on their own.

**Path C — no kanban toolset, dispatcher not requested:**

Use Hermes' `todo_list` tool if it's in your tool list — it's a real fallback when the plugin is loaded, not "no list tool at all." Otherwise, enumerate the plan with the same tags in your own working notes and pass the relevant items into each `delegate_task` call's `context` in Step 4 — the plan lives in what you tell each child.

Use the same tags the other platform skills use, so cards/notes read consistently across the project:
- `[implement]` — Surgeon (you)
- `[review]` — Copilot
- `[test]` — Tester
- `[research]` — Language Lawyer (if spawned)
- `[docs]` — Editor (if spawned)
- `[tooling]` — Toolsmith (if spawned)
- `[structure]` — Program Clerk (if spawned)

## Step 4: Spawn the Team

**Greenfield rule:** if there's no implementation yet (empty or near-empty tree), build it yourself first. Spawn Copilot/Tester only once there's an actual file or diff for them to check — spawning either onto an empty tree burns a call and reviews nothing. The order that works: implement → spawn review/test → fix from findings.

Hermes has no persistent per-role agent-definition file the way Codex's `.codex/agents/*.toml`, OpenCode's `.opencode/agents/*.md`, or Grok's `.grok/agents/*.md` do. `delegate_task` is fully ephemeral: `goal`, `context`, and `output_schema` are passed at call time only, with no on-disk registry to point at. Hermes "profiles" do provide named, persistent identities, but each is a full separate Hermes home directory (own config, skills, memory, state db) — too heavy to stand in for a lightweight role file here.

Spawn each role with `delegate_task`, and the target skill's full body pasted into `context` — since there's no named agent to reference, the skill content itself is the role contract:

```
delegate_task(
  goal: "Review the [FEATURE] implementation against its design intent; report Blocking/Important/Suggestions findings.",
  context: "<paste skills/copilot/SKILL.md body>\n\nProject: [one-paragraph summary]. Diff/files to review: [...]. Do not edit source.",
  output_schema: { ... }
)
```

(Some documentation mentions a `role` field on `delegate_task` — omit it unless your Hermes version's live tool definition actually lists it; `goal`/`context`/`output_schema` are the parameters that are field-verified to work.)

Repeat per selected role: `tester` → `skills/tester/SKILL.md`, `language-lawyer` → `skills/language-lawyer/SKILL.md`, `editor` → `skills/editor/SKILL.md`, `toolsmith` → `skills/toolsmith/SKILL.md`, `program-clerk` → `skills/program-clerk/SKILL.md`. Give each child its full task up front (project summary, the card it owns, file ownership, the concrete diff/spec/question) — there is no lightweight-first-turn/resume split to rely on.

If the artifact needs a CDN-hosted library, tell each child that "Cloudflare" means a real `*.cloudflare.com` URL (e.g. cdnjs) — if a library isn't actually hosted there, inline it or skip it. Don't let a child invent a URL.

`delegation.max_concurrent_children` (default 10, set in the user's `~/.hermes/config.yaml`, not a repo file) governs how many of these run at once. Once there's something to review, issue all selected roles' `delegate_task` calls together rather than waiting on each in turn.

**Reconciling with the kanban board (if you have one):** `delegate_task` children are not spawned by the kanban dispatcher. No Hermes doc directly addresses whether they inherit your profile's orchestrator-side `kanban` toolset (`kanban_create`/`kanban_list`/`kanban_link`/`kanban_unblock`). The general subagent-inheritance rule (children inherit the parent's enabled toolsets, with only `delegate_task`, `clarify`, `memory`, `send_message`, and `cronjob` named as blocked — `kanban` isn't on that list) suggests it likely could. Don't rely on a child being unable to touch the board either way — tell it explicitly not to call `kanban_create`/`kanban_link`/`kanban_unblock` even if they show up in its tool list. What a `delegate_task` child definitely does not get is the separate `HERMES_KANBAN_TASK`-gated worker tools (`kanban_show`, `kanban_complete`, `kanban_block`, `kanban_request_review`, `kanban_comment`, etc. — those exist only for a task the dispatcher itself launched, whatever the assignee). So even with the orchestrator toolset enabled, nothing in the default `delegate_task` flow can mark a card `done`, `blocked`, or commented-on directly. A Path B card the dispatcher itself actually launches — any assignee, a shared `default` profile included, not only a dedicated per-role one — is the exception: it gets `HERMES_KANBAN_TASK` and the full lifecycle, and closes its own card.

Treat the board as **plan-of-record, not live status** for anything not covered by that exception: create the cards up front for visibility and dependency tracking, `kanban_list` to check them, and `kanban_unblock` a card once its `delegate_task` result confirms it's actually done.

### File Ownership Rules (Include in Every `delegate_task` Context)

- **Copilot**: read-only across the repo. Never edits source.
- **Tester**: owns `tests/`, `**/*.test.*`, `**/*.spec.*`, and test utilities. Assert only observable spec behavior — don't let a test lock in an incidental implementation detail (e.g., one specific UI dismiss gesture) that a legitimate fix would then have to break to pass Copilot's review.
- **Language Lawyer**: read-only research. May search the web. Never edits files.
- **Editor**: owns `docs/`, `README*`, `*.md` (except code-adjacent notes).
- **Toolsmith**: owns `scripts/`, tooling configs, Makefiles, new automation. Every tool needs `--help` and tests.
- **Program Clerk**: proposes structure changes only; executes renames/refactors only after Surgeon approval.

No worktree isolation is documented for `delegate_task` children — these rules are enforced by convention and Surgeon review, the same as on OpenCode and Copilot CLI, not by a platform-level sandbox. Never run two roles with overlapping ownership at once, and never run two same-file `[implement]` cards concurrently — chain them (see Step 3, Path B).

## Step 5: Handoff

If your harness backgrounds `delegate_task` children, give the user a one-line status here and stop — don't poll for completion. When a child's result comes back, treat any self-reported outcome (e.g. "33/33 passed") as a claim, not a fact: re-run the suite yourself before telling the user it's done. A child can fail at the transport level (e.g. a broken pipe) minutes after doing real work, with its own success claim being the only thing that came back.

Then tell the user:

> Team is live. You are the Surgeon: continue the critical path while your teammates run as `delegate_task` children.
>
> [If a Path B card is actually running under the dispatcher (gateway or `hermes kanban dispatch` active) — any assignee, `default` included:] That worker closes its own card (`kanban_complete`) — `kanban_list()` reflects live status.
>
> [Otherwise, if the kanban toolset is enabled:] The plan is on the kanban board (`kanban_list()` any time for state — durable, survives this session ending), but it's plan-of-record only — neither your teammates nor this session can mark a card `done` or comment on it, since those tools are dispatcher-gated. Treat each `delegate_task` result as the real completion signal, and use `kanban_unblock` to release anything it was blocking.
>
> [If there's no board at all:] There's no shared task-list tool here — the plan lives in what you told each teammate (or your `todo_list`, if you have one). Track completion from each `delegate_task` result as it returns.
>
> No worktree isolation here — respect the file-ownership rules above; never let two roles touch the same files.
>
> Run `assemble-team` any time for the full role reference.

## Optional: Re-Review Loop (Deferred)

Field notes proposed an automatic loop: after Copilot/Tester return, enqueue their Blocking + Important findings and any failing tests as new `[implement]` cards, re-dispatch Copilot/Tester once fixed, and ignore Suggestions unless the user asks for them. That's not wired into the steps above — deciding whether and how to automate it (kanban-only, since a `delegate_task` round has no card to re-enqueue against) is deferred to a future revision. For now, do it by hand: read the findings, fix them yourself or spawn a fresh `delegate_task` round, and only loop back to Copilot/Tester when there's something new to check.

## Advanced: fully autonomous kanban workers (optional)

If dedicated Hermes profiles already exist for these roles (`~/.hermes/profiles/<role>/`), the kanban dispatcher can spawn and run those profiles against `kanban_create`d cards directly — this is the same worker lifecycle Path B already gets even against a shared `default` profile (any dispatcher-launched card gets `HERMES_KANBAN_TASK` and the full `kanban_show`/`kanban_complete`/`kanban_block`/`kanban_request_review` toolset). What dedicated profiles add is a persistent, named identity per role instead of everyone sharing `default`. Setting this up means a full separate Hermes home directory per role, which is out of scope for a quick team assembly; consider it only if Hermes teams are run often enough to justify it.

## Anti-Patterns to Prevent

- Do not spawn the full team for tiny changes.
- Do not spawn Copilot/Tester onto an empty tree — implement first, review/test second.
- Do not let two roles write the same files, and do not run two same-file `[implement]` cards concurrently.
- Do not assign a kanban card to a role name that isn't an actual Hermes profile — it will silently never spawn a worker.
- Do not trust a child's self-reported test results without re-running the suite yourself.
- Do not tell the user a card (or its underlying work) is done without checking the actual `delegate_task` result — the board itself won't reflect it, except for a Path B card the dispatcher itself actually launched (any assignee, `default` included).
- Do not assume a `delegate_task` child, or the Surgeon session, can mark a kanban card `done`/`blocked` or comment on it — those tools are dispatcher-gated; only a dedicated profile-based worker gets them.
