---
name: maggus-vision
description: Create or improve a VISION.md for a software project through guided, iterative dialogue. Use when starting a new project or when asked to write, improve, or review a vision or VISION.md.
user-invocable: true
disable-model-invocation: true
---

# Vision Writer

Create or enhance a `VISION.md` through iterative dialogue — asking targeted questions, writing drafts, and refining until nothing important is missing.

**This skill writes to `VISION.md`.** It will update or create the file after the user confirms.

---

## Phase 1: Check for Existing Vision

First, check whether a `VISION.md` already exists in the current working directory.

**If it exists:**
- Read it in full
- Summarise what is already covered (2–3 sentences)
- Identify the most obvious gaps or underdefined areas upfront
- Tell the user: *"I found an existing VISION.md. I'll enhance it rather than replace it. Here's what's already covered: … Here's what looks incomplete: …"*
- Skip straight to Phase 4 (Gap Review), using the existing content as the baseline

**If it does not exist:**
- Proceed with Phase 1: Seed below

---

## Phase 1: Seed (new files only)

Ask the user for a one-sentence description of the product:

> "Give me a one-liner: what does this product do, and who is it for?"

Then ask for the tech stack if not already known (language, framework, any key infrastructure choices). Keep this brief — just enough to anchor the document.

---

## Phase 2: Core Concepts

Identify the main entities and flows in the product. For each concept the user mentions, drill into:

- **What is it?** — clear definition
- **What does it do / what rules govern it?** — behaviors, constraints, edge cases
- **How does it relate to other concepts?** — ownership, dependencies, cardinality

Ask one focused follow-up question at a time. Do not ask multiple questions in a single message. Wait for the answer before proceeding.

Typical concepts to probe (adapt to the product):

- The primary actor (user, agent, device, etc.) and how they are identified/authenticated
- The core resource they operate on
- Multi-tenancy — is there an organisation/tenant concept? How are they created?
- Any real-time or async communication between components
- Data lifecycle — what gets stored, where, for how long
- Concurrency / locking — can multiple actors touch the same resource?
- Failure modes — what happens when something goes wrong?
- Notifications — how does the system communicate events to users?

---

## Phase 3: First Draft

After covering the core concepts, write a complete `VISION.md` draft. Use this structure as a guide — adapt sections to fit the product:

```
# <Product Name>

<one-paragraph overview>

## Core Concepts

### <Concept 1>
...

### <Concept 2>
...

## <Other top-level sections as needed>
(e.g. Organisations & Subscriptions, Notifications, AI Integration, Frontend, Dashboard Metrics)
```

Present the draft inline. Do not write the file yet.

---

## Phase 4: Gap Review

After presenting the draft, review it critically for missing decisions. Ask about any of the following that are not yet answered:

- **Error/failure policy** — what happens on failure? retries? manual intervention?
- **Stale/timeout behavior** — what happens when something goes offline or is abandoned?
- **Ordering and concurrency** — sequential or parallel? dependencies between items?
- **Import/export formats** — if data enters or leaves the system, what is the format?
- **Extensibility points** — anything that should be configurable or swappable (AI provider, notification channel, auth provider)?
- **Subscription / billing shape** — even if details are deferred, is there a free tier? seat limits?
- **Roles and permissions** — are there different user roles within the product?

Ask one gap at a time. If the user says "defer that", note it explicitly in the document with a comment like `— details to be defined`.

---

## Phase 5: Iterate

Incorporate the user's answers and present an updated draft. Repeat Phase 4 until the user confirms the vision is complete.

Ask: *"Is there anything else missing, or shall I write the file?"*

---

## Phase 6: Write the File

Once the user confirms:

- **If enhancing an existing file** — apply only the changes needed. Preserve all existing content that is still accurate; update or extend sections that changed; add new sections for gaps that were filled. Do not rewrite sections that don't need it.
- **If creating a new file** — write the full content.

Confirm when done: *"VISION.md updated. Ready to move on to ARCHITECTURE.md when you are."*

---

## Style Guidelines

- Use **bold** for key terms on first introduction
- Use bullet lists for behaviors and constraints — not prose paragraphs
- Prefer concrete, specific language — avoid vague words like "handles", "manages", "supports"
- Explicitly call out defaults for configurable values (e.g. "default: 2 hours")
- Mark deferred decisions clearly rather than omitting them
- Keep sections focused — if a section grows beyond ~10 bullets, consider splitting it
