---
name: maggus-plan
description: "Generate a feature implementation plan for one or more new features. Use when planning a feature, starting a new project, or when asked to create an implementation plan. Triggers on: create a feature, plan a feature, write feature plan, plan this feature"
user-invocable: true
---

# Feature Implementation Plan Generator

Create detailed, actionable feature plans that are architecture-aware and suitable for parallel agent execution.

---

## The Job

1. Load project context (VISION.md, ARCHITECTURE.md, PROJECT_CONTEXT.md)
2. Receive a feature description from the user
3. Ask 3-5 essential clarifying questions (with lettered options)
4. Generate a structured feature plan based on answers
5. Iterate: resolve all open questions with the user until none remain
6. Save to `.maggus/features/feature_NNN.md`

**Important:** Do NOT start implementing. Just create the feature plan.

---

## Step 0: Load Project Context

Before asking any questions, read what already exists:

1. **Check for `VISION.md`** in the current directory — if found, read it in full. The vision defines the product's purpose, core concepts, and boundaries. Every feature must align with it.
2. **Check for `ARCHITECTURE.md`** in the current directory — if found, read it in full. The architecture defines how the system is built. Every task must respect its constraints, patterns, and component boundaries.
3. **Check for existing features in `.maggus/features/`** — scan for `feature_*.md` files to determine the next rolling number and avoid duplicate or conflicting work.

Then tell the user what you found:

- **Both files found** → *"I've loaded your VISION.md and ARCHITECTURE.md. I'll design this feature to align with your vision and respect the existing architecture."* Briefly state the most relevant architectural constraints for the feature at hand (e.g. "Your architecture uses actor-based concurrency, so I'll design tasks around that pattern.").
- **Only VISION.md** → *"I found your VISION.md but no ARCHITECTURE.md. I'll align with the vision. Consider running `/maggus-architecture` to define the technical architecture."*
- **Only ARCHITECTURE.md** → *"I found your ARCHITECTURE.md but no VISION.md. I'll respect the architecture. Consider running `/maggus-vision` to define the product vision."*
- **Neither found** → *"No VISION.md or ARCHITECTURE.md found. I'll proceed based on your description. Consider running `/maggus-vision` and `/maggus-architecture` to establish project foundations."*

**Throughout the entire plan generation, keep VISION.md and ARCHITECTURE.md in mind:**
- Every goal must trace back to the vision
- Every task must respect architectural boundaries (component ownership, data flow patterns, API contracts)
- Flag any feature requirements that conflict with the current architecture
- Note when a feature may require architecture updates (and suggest updating ARCHITECTURE.md)

---

## Step 1: Clarifying Questions

Ask only critical questions where the initial prompt is ambiguous. Focus on:

- **Problem/Goal:** What problem does this solve? (Relate to vision goals)
- **Core Functionality:** What are the key actions?
- **Scope/Boundaries:** What should it NOT do?
- **Success Criteria:** How do we know it's done?
- **Architecture Fit:** Does this introduce new components or extend existing ones? (Relate to ARCHITECTURE.md)

### Format Questions Like This:

```
1. What is the primary goal of this feature?
   A. Improve user onboarding experience
   B. Increase user retention
   C. Reduce support burden
   D. Other: [please specify]

2. Who is the target user?
   A. New users only
   B. Existing users only
   C. All users
   D. Admin users only

3. What is the scope?
   A. Minimal viable version
   B. Full-featured implementation
   C. Just the backend/API
   D. Just the UI
```

This lets users respond with "1A, 2C, 3B" for quick iteration. Remember to indent the options.

---

## Step 2: Feature Plan Structure

Generate the plan with these sections:

### 1. Introduction/Overview
Brief description of the feature and the problem it solves.

**Include an "Architecture Context" subsection** that summarizes:
- Which existing components/services this feature touches (from ARCHITECTURE.md)
- How this feature aligns with the product vision (from VISION.md)
- Any new components or patterns this feature introduces

### 2. Goals
Specific, measurable objectives (bullet list). Each goal should trace back to a vision objective where applicable.

### 3. Tasks

Each task needs:
- **Title:** Short descriptive name
- **Description:** "As a [user], I want [feature] so that [benefit]"
- **Acceptance Criteria:** Verifiable checklist of what "done" means
- **Token Estimate:** Estimated token usage as a concrete number (e.g. `~40k tokens`)
- **Predecessors:** Which tasks must complete before this one can start (by task ID), or `none`
- **Successors:** Which tasks depend on this one (by task ID), or `none`
- **Parallel:** Whether this task can run in parallel with other tasks that share no dependency
- **Model (optional):** Recommended model if the task benefits from a specific one

Each task should be small enough to implement in one focused session.

**Task ID format:** `TASK-NNN-XXX` where `NNN` is the feature number and `XXX` is the task number within the feature. For example, feature 3 would have tasks `TASK-003-001`, `TASK-003-002`, etc.

**Format:**
```markdown
### TASK-NNN-001: [Title]
**Description:** As a [user], I want [feature] so that [benefit].

**Token Estimate:** ~50k tokens
**Predecessors:** none
**Successors:** TASK-NNN-002, TASK-NNN-003
**Parallel:** yes — can run alongside TASK-NNN-004
**Model:** (optional, e.g. `opus` for complex logic, `sonnet` for straightforward CRUD, `haiku` for simple file changes)

**Acceptance Criteria:**
- [ ] Specific verifiable criterion
- [ ] Another criterion
- [ ] Typecheck/lint passes
- [ ] **[Backend tasks only]** Unit tests are written and successful
- [ ] **[UI tasks only]** Verify in browser using dev-browser skill
```

#### Token Estimate Guidelines

Estimate the token usage as a concrete number (e.g. `~30k tokens`). Use these reference points:

- **~10k–25k** — Simple file changes, config updates, small utility functions
- **~25k–75k** — Single component/service, moderate logic, tests included
- **~75k–150k** — Multi-file feature, complex logic, multiple test cases
- **~150k–300k** — Cross-cutting concern, major refactor, extensive test coverage

Consider: lines of code to write/modify, number of files touched, test complexity, and how much existing code the agent needs to read for context. Be specific — `~35k` is better than `~50k` if you can justify it.

#### Model Recommendations

Only specify a model when it meaningfully matters:

- **`opus`** — Complex architectural decisions, intricate business logic, multi-file refactors requiring deep context
- **`sonnet`** — Standard feature work, CRUD operations, well-defined tasks (good default — omit the field if sonnet is fine)
- **`haiku`** — Simple repetitive tasks, config changes, boilerplate generation, formatting

If no model is specified, the executor should use their default.

#### Dependency & Parallelism Rules

- Tasks with `Predecessors: none` can start immediately
- Tasks are parallelizable when they have no shared predecessors AND touch different components/files
- Always mark `Parallel: yes/no` explicitly — the executor relies on this for agent dispatching
- Keep the dependency graph as shallow as possible to maximize parallelism

**Important:**
- Acceptance criteria must be verifiable, not vague. "Works correctly" is bad. "Button shows confirmation dialog before deleting" is good.
- **For any task with UI changes:** Always include "Verify in browser using dev-browser skill" as acceptance criteria.
- **Respect ARCHITECTURE.md boundaries:** Tasks should align with the component ownership defined in the architecture. If a task spans multiple components, split it or note the cross-cutting nature.

### 4. Task Dependency Graph

Include a visual summary of the task execution order:

```
TASK-NNN-001 ──→ TASK-NNN-003 ──→ TASK-NNN-005
TASK-NNN-002 ──→ TASK-NNN-004 ──┘
```

Also include a summary table:

```markdown
| Task | Estimate | Predecessors | Parallel | Model |
|------|----------|--------------|----------|-------|
| TASK-NNN-001 | ~50k | none | yes (with 002) | — |
| TASK-NNN-002 | ~15k | none | yes (with 001) | haiku |
| TASK-NNN-003 | ~100k | 001 | no | opus |
| TASK-NNN-004 | ~40k | 002 | no | — |
| TASK-NNN-005 | ~20k | 003, 004 | no | — |
```

**Total estimated tokens:** Sum of all task estimates.

### 5. Functional Requirements
Numbered list of specific functionalities:
- "FR-1: The system must allow users to..."
- "FR-2: When a user clicks X, the system must..."

Be explicit and unambiguous.

### 6. Non-Goals (Out of Scope)
What this feature will NOT include. Critical for managing scope.

### 7. Design Considerations (Optional)
- UI/UX requirements
- Link to mockups if available
- Relevant existing components to reuse (reference ARCHITECTURE.md)

### 8. Technical Considerations (Optional)
- Known constraints or dependencies from ARCHITECTURE.md
- Integration points with existing systems
- Performance requirements
- Whether ARCHITECTURE.md needs updating after this feature

### 9. Success Metrics
How will success be measured?
- "Reduce time to complete X by 50%"
- "Increase conversion rate by 10%"

### 10. Open Questions
Remaining questions or areas needing clarification.

---

## Step 3: Refinement Loop

After generating the initial plan, **do not save it yet.** Instead, iterate until all open questions are resolved:

1. **Present the plan** to the user, including the Open Questions section.
2. **Walk through each open question** one at a time, asking the user for their decision. Format as lettered options where possible (same style as Step 1).
3. **Incorporate answers** into the plan — update tasks, acceptance criteria, functional requirements, or scope as needed. Resolving a question may:
   - Add, remove, or modify tasks (update the dependency graph accordingly)
   - Change token estimates or model recommendations
   - Move items between Goals and Non-Goals
   - Surface new technical considerations
4. **Check for new questions** — sometimes resolving one question reveals another. If new questions arise, add them and continue the loop.
5. **Repeat** until the Open Questions section is empty or the user explicitly defers remaining items (mark deferred items with `— deferred`).
6. **Final confirmation** — Present the final plan and ask: *"All open questions are resolved. Ready to save?"*

Only after the user confirms, proceed to save the file.

**Important:** Each iteration should show only the changes, not the full plan again — unless the user asks to see the full plan. Keep the loop lightweight.

---

## Writing for Junior Developers

The feature plan reader may be a junior developer or AI agent. Therefore:

- Be explicit and unambiguous
- Avoid jargon or explain it
- Provide enough detail to understand purpose and core logic
- Number requirements for easy reference
- Use concrete examples where helpful

---

## Output

- **Format:** Markdown (`.md`)
- **Location:** `.maggus/features/`
- **Filename:** `feature_NNN.md` where NNN is a zero-padded rolling number (e.g. `feature_001.md`, `feature_002.md`)

To determine the next number, check existing `feature_*.md` files in `.maggus/features/` and increment.

---

## Example Plan

```markdown
# Feature 001: Leberkassemme

## Introduction

Build a traditional Bavarian Leberkassemme — a warm slice of freshly baked Leberkäs served in a crispy Semmel (bread roll), optionally with sweet mustard. This is the quintessential Bavarian street food, found at every Metzger (butcher) and Imbiss across Bavaria.

### Architecture Context

- **Vision alignment:** Fulfils the core product goal of "authentic Bavarian street food experience"
- **Components involved:** Kitchen backend (meat processing), Bakery service (Semmel supply), Assembly frontend (serving counter)
- **New patterns:** Introduces temperature monitoring (meat thermometer integration)

## Goals

- Produce a Leberkäs with a crispy golden-brown crust and a smooth, juicy interior
- Serve on a fresh Semmel that holds up structurally without going soggy
- Achieve the classic flavor balance of savory meat, crusty bread, and sweet mustard
- Keep preparation approachable for a home kitchen

## Tasks

### TASK-001-001: Prepare the Leberkäs Brät
**Description:** As a cook, I want to prepare the Leberkäs meat mixture (Brät) so that it has the right smooth, emulsified texture.

**Token Estimate:** ~50k tokens
**Predecessors:** none
**Successors:** TASK-001-002
**Parallel:** yes — can run alongside TASK-001-003

**Acceptance Criteria:**
- [ ] Pork, beef, and back fat are finely ground (twice through the finest plate)
- [ ] Onion is finely diced and mixed in
- [ ] Brät is seasoned with salt, pepper, marjoram, and a pinch of nutmeg
- [ ] Mixture is smooth and paste-like with no visible chunks
- [ ] Ice water added during mixing to keep temperature below 12°C

### TASK-001-002: Bake the Leberkäs
**Description:** As a cook, I want to bake the Leberkäs loaf so that it develops a crispy crust while staying juicy inside.

**Token Estimate:** ~40k tokens
**Predecessors:** TASK-001-001
**Successors:** TASK-001-004
**Parallel:** no — requires Brät from TASK-001-001
**Model:** opus — temperature timing is critical

**Acceptance Criteria:**
- [ ] Brät is pressed into a greased loaf pan (Kastenform), slightly domed on top
- [ ] Top is scored in a diamond pattern and brushed with water
- [ ] Oven preheated to 180°C (convection)
- [ ] Baked for approximately 60–75 minutes until core temperature reaches 69°C
- [ ] Crust is golden-brown and slightly cracked along the score lines

### TASK-001-003: Source or bake the Semmeln
**Description:** As a cook, I want fresh Semmeln so that the bread is crusty outside and soft inside.

**Token Estimate:** ~15k tokens
**Predecessors:** none
**Successors:** TASK-001-004
**Parallel:** yes — can run alongside TASK-001-001, TASK-001-002
**Model:** haiku — straightforward procurement

**Acceptance Criteria:**
- [ ] Semmeln are fresh from the bakery or baked same-day
- [ ] Crust is crispy and audibly crunches when pressed
- [ ] Interior is soft and airy, not dense or stale
- [ ] Size is appropriate — roughly 10 cm diameter

### TASK-001-004: Assemble the Leberkassemme
**Description:** As a hungry Bavarian, I want a properly assembled Leberkassemme so that every bite has the right ratio of bread, meat, and mustard.

**Token Estimate:** ~20k tokens
**Predecessors:** TASK-001-002, TASK-001-003
**Successors:** none
**Parallel:** no — requires outputs from both predecessor tasks

**Acceptance Criteria:**
- [ ] Semmel is sliced open horizontally (not all the way through)
- [ ] One thick slice of Leberkäs (approx. 1–1.5 cm) is cut fresh from the loaf while still warm
- [ ] Leberkäs slice fits the Semmel — trimmed or folded if needed, no tiny slice in a huge roll
- [ ] Sweet mustard (süßer Senf) applied generously on one side
- [ ] Served immediately while Leberkäs is still warm

## Task Dependency Graph

```
TASK-001-001 ──→ TASK-001-002 ──→ TASK-001-004
TASK-001-003 ────────────────────┘
```

| Task | Estimate | Predecessors | Parallel | Model |
|------|----------|--------------|----------|-------|
| TASK-001-001 | ~50k | none | yes (with 003) | — |
| TASK-001-002 | ~40k | 001 | no | opus |
| TASK-001-003 | ~15k | none | yes (with 001) | haiku |
| TASK-001-004 | ~20k | 002, 003 | no | — |

**Total estimated tokens:** ~125k

## Functional Requirements

- FR-1: Leberkäs Brät must be emulsified to a smooth consistency — no coarse mince texture
- FR-2: Baking must produce a crust that is visibly golden-brown and slightly crisp
- FR-3: Core temperature of the finished Leberkäs must reach at least 69°C
- FR-4: Semmel must be fresh — maximum 4 hours old at time of serving
- FR-5: Assembly must happen immediately after slicing the Leberkäs to preserve warmth
- FR-6: Sweet mustard (Händlmaier's or equivalent) must be available as condiment

## Non-Goals

- No Weißwurst, no Brezen — this is about the Leberkassemme only
- No fancy toppings like lettuce, tomato, or cheese (this is not a burger)
- No Pizzaleberkäs or Käsleberkäs variants in this iteration
- No deep-frying or pan-frying — oven-baked only

## Technical Considerations

- Use a meat thermometer to verify core temperature — guessing is not acceptable
- Keep Brät cold during preparation to prevent fat separation
- A Kastenform (loaf pan) of approx. 30 cm works for a standard batch
- If no Metzger-quality grinder is available, ask your local Metzger to grind the meat twice on the finest setting
- **Architecture note:** Temperature monitoring pattern introduced here should be documented in ARCHITECTURE.md for reuse

## Success Metrics

- Crust is crispy and golden — passes the visual and crunch test
- Interior is smooth, juicy, and uniformly pink — no gray spots or air pockets
- Semmel-to-Leberkäs ratio feels balanced in every bite
- At least one person says "Des is a guade Leberkassemme!"

## Open Questions

- Should we offer a Spiegelei (fried egg) on top as an optional upgrade?
- Is homemade sweet mustard worth the effort, or is Händlmaier's sufficient?
```

---

## Checklist

Before saving the feature plan:

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
- [ ] User confirmed final plan
- [ ] Saved to `.maggus/features/feature_NNN.md`
