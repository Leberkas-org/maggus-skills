---
name: bryan-plan
description: "Generate a feature implementation plan and save it to Bryan. Requires a Bryan subscription and configured MCP server. Use when planning a feature with Bryan as the backend. Triggers on: bryan plan, create feature in bryan, plan feature bryan"
user-invocable: true
---

# Bryan Feature Plan

Follow the `maggus-plan` skill exactly — same questions, same plan structure, same refinement loop.

**The only difference is the Output step.** Instead of writing a file, save to Bryan via MCP.

---

## Output

Call `mcp__bryan__create_feature` with the following mapping:

| Plan field | MCP parameter |
|---|---|
| Feature title | `title` |
| `git remote get-url origin` | `repositoryOriginUrl` |
| Introduction text | `introduction` |
| Goals list | `goals` (string[]) |
| Functional requirements | `functionalRequirements` (string[]) |
| Non-goals | `nonGoals` (string[]) |
| Technical Considerations + dependency graph (as text) | `technicalConsiderations` |
| Success metrics | `successMetrics` (string[]) |
| Tasks | `tasks` (TaskInput[]) |

**TaskInput mapping per task:**

| Plan field | TaskInput field |
|---|---|
| Task title | `title` |
| Description | `description` |
| Acceptance criteria items | `acceptanceCriteria` (string[]) |
| Token estimate (e.g. `"50k"`) | `tokenEstimate` |
| Predecessor task **titles** | `predecessorTitles` (string[]) |
| Model | `model` |

Notes:
- `repositoryOriginUrl`: run `git remote get-url origin` to get the value
- `predecessorTitles`: use the exact task title strings — Bryan resolves them to IDs after creation
- `successors` and `parallel` are planning-only concepts; include them in the dependency graph text inside `technicalConsiderations`
- Bryan assigns the feature ID (e.g. `F-1`) — no manual numbering needed
- The response includes `featureId` and `predecessorWarnings` — surface any warnings to the user

## Checklist

Before saving:

- [ ] Loaded and reviewed VISION.md (if exists)
- [ ] Loaded and reviewed ARCHITECTURE.md (if exists)
- [ ] Feature aligns with vision goals
- [ ] Tasks respect architecture boundaries
- [ ] Asked clarifying questions with lettered options
- [ ] Incorporated user's answers
- [ ] Tasks are small and specific
- [ ] Every task has token estimate, predecessors, successors, and parallel flag
- [ ] Model specified only where it meaningfully matters
- [ ] Dependency graph is as shallow as possible for maximum parallelism
- [ ] Functional requirements are numbered and unambiguous
- [ ] Non-goals section defines clear boundaries
- [ ] Flagged any architecture updates needed
- [ ] All open questions resolved or explicitly deferred by user
- [ ] Called `mcp__bryan__create_feature` and received a feature ID
- [ ] Surfaced any `predecessorWarnings` to the user
