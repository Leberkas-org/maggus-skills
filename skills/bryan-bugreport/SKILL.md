---
name: bryan-bugreport
description: "Create a structured bug ticket and save it to Bryan. Requires a Bryan subscription and configured MCP server. Use when reporting a bug with Bryan as the backend. Triggers on: bryan bug, report bug in bryan, file bug bryan"
user-invocable: true
---

# Bryan Bug Report

Follow the `maggus-bugreport` skill exactly — same investigation, same diagnostic questions, same ticket structure.

**The only difference is the Output step.** Instead of writing a file, save to Bryan via MCP.

---

## Output

Call `mcp__bryan__create_bug` with the following mapping:

| Bug ticket field | MCP parameter |
|---|---|
| Title | `title` |
| What currently happens | `currentBehavior` |
| What should happen | `expectedBehavior` |
| Steps to Reproduce | `stepsToReproduce` (string[]) |
| Severity | `severity` (`low`/`medium`/`high`/`critical`) |
| User story tasks | `tasks` (TaskInput[]) |

**TaskInput mapping per user story:**

| Field | TaskInput field |
|---|---|
| Task title | `title` |
| Description | `description` |
| Acceptance criteria items | `acceptanceCriteria` (string[]) |
| Predecessor task **titles** | `predecessorTitles` (string[]) |

Notes:
- Bryan assigns the bug ID (e.g. `BUG-1`) — no manual numbering needed
- The response includes `bugId` and `predecessorWarnings` — surface any warnings to the user

## Checklist

Before saving:

- [ ] Investigated the codebase (read relevant files, checked git history)
- [ ] Asked diagnostic questions only for things you couldn't determine
- [ ] Incorporated user's answers
- [ ] Root cause section references specific files and line numbers
- [ ] Tasks are small and have verifiable acceptance criteria
- [ ] Every task includes regression check and test criteria
- [ ] Called `mcp__bryan__create_bug` and received a bug ID
- [ ] Surfaced any `predecessorWarnings` to the user
