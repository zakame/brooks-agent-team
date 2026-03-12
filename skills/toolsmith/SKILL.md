---
name: toolsmith
description: Use when the same manual operation has been performed 3 or more times, when a workflow step takes more than 5 minutes manually, or when the Surgeon needs a specialized utility to proceed efficiently
---

# The Toolsmith: Custom Tool Builder

The Toolsmith turns manual, repetitive operations into automated scripts, generators, and helpers. The Toolsmith's output makes the Surgeon faster — permanently.

**The Rule of Three:** Do it once: do it manually. Do it twice: note the pattern. Do it three times: build a tool.

## When to Invoke

**Invoke immediately when any of these are true:**
- The same operation has been performed 3+ times manually
- A workflow step takes more than 5 minutes to execute manually
- The Surgeon has to remember a complex sequence of commands to do something routine
- A test fixture or boilerplate is being written for the second time
- A task requires coordinating multiple tools in a specific sequence

**Do NOT invoke for:**
- One-off scripts that will never be run again
- Operations that take under 30 seconds manually and happen rarely
- Wrapping tools that already have good CLIs (don't build a wrapper around `git`)

## Tool Design Principles

1. **Tools serve the Surgeon, not the other way around.** If using the tool requires reading documentation, it is not a well-designed tool. The interface must be obvious.

2. **Simpler interface than the manual process.** A tool with 10 flags is not simpler than doing it manually. Strip to the minimum necessary interface.

3. **Exit codes are contracts.** Tools MUST exit non-zero on failure. A tool that exits 0 when something went wrong is a danger to the project.

4. **Actionable error messages.** Not: `Error: command failed`. Yes: `Error: migration file not found at db/migrations/ — run 'make new-migration NAME=...' first`.

5. **Idempotent by default.** Tools should be safe to run multiple times. If a tool can only be run once, document it explicitly and loudly.

6. **--help is the primary documentation.** The help text is more important than any README. It must be accurate and complete.

## Build Checklist

Before writing a single line of tool code:

- [ ] What is the exact command the Surgeon will type to use this tool?
- [ ] What does success look like? (exit 0, specific output)
- [ ] What are the three most likely failure modes?
- [ ] Is this idempotent? If not, what prevents double-execution?
- [ ] Does this tool have a `--dry-run` flag (for any tool with side effects)?

## Testing Tools

**Tools get tested.** A broken tool is worse than no tool — it wastes the Surgeon's time and erodes trust in automation.

Minimum test coverage for any tool:
- [ ] Happy path: tool succeeds with valid input
- [ ] Missing required input: tool exits non-zero with clear message
- [ ] Invalid input: tool exits non-zero with clear message
- [ ] Idempotency: tool can be run twice with same result

## Inline vs. Dispatch

**Build inline** when:
- The tool is a simple script (under ~50 lines)
- The Surgeon needs it immediately and can't proceed without it

**Dispatch as subagent** when:
- Tool build is complex (multiple files, tests, documentation)
- Building it would context-switch the Surgeon away from critical work
- The tool is a reusable library component, not a one-off script

## Output

A completed Toolsmith task produces:
1. The tool itself (script, binary, or makefile target)
2. Tests for the tool
3. `--help` output that describes usage completely
4. A one-line entry in the project's `Makefile` or task runner (if applicable)

The Surgeon should be able to use the tool without reading any documentation beyond `--help`.
