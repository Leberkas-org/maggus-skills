---
name: architecture
description: Create or improve an ARCHITECTURE.md for a software project through guided, iterative dialogue. Use when starting a new project or when asked to write, improve, or review an architecture or ARCHITECTURE.md.
user-invocable: true
disable-model-invocation: true
---

# Architecture Writer

Create or enhance an `ARCHITECTURE.md` through iterative dialogue — deriving technical decisions from the vision, asking targeted questions, and refining until the architecture is complete and consistent.

**This skill writes to `ARCHITECTURE.md`.** It will update or create the file after the user confirms.

---

## Phase 1: Load Context

Before asking any questions, read what already exists:

1. Check for `VISION.md` in the current directory — if found, read it in full. The vision is the primary source of truth; the architecture must implement everything described there.
2. Check for `ARCHITECTURE.md` — if found, read it in full.

Then tell the user what you found and what mode you are in:

- **No ARCHITECTURE.md** → *"I'll create a new ARCHITECTURE.md based on your VISION.md."* Proceed to Phase 1b.
- **ARCHITECTURE.md exists** → *"I found an existing ARCHITECTURE.md. I'll enhance it. Here's what's already covered: … Here's what looks incomplete or inconsistent with the vision: …"* Skip to Phase 4 (Gap Review).
- **No VISION.md and no ARCHITECTURE.md** → Ask the user for a brief product description, then proceed to Phase 1b.

---

## Phase 1b: Define System Shape

Before deriving anything, establish which high-level components the system has. Try to infer this from the vision or existing architecture — if it is clear, state your inference and ask the user to confirm. If it is not clear, ask explicitly:

> "Which components does this system include?"
> A. Backend only — API/service with no UI
> B. Frontend only — client app with no custom server-side logic
> C. Full-stack — backend + frontend
> D. Monolith — single deployable that renders views server-side
> E. Other — [please describe]

Record the answer as the **system shape**. All subsequent phases use it to decide which sections to include or skip:

| Component | Include when shape is |
|---|---|
| API Layer | Backend, Full-stack |
| Backend section | Backend, Full-stack, Monolith |
| Frontend section | Frontend, Full-stack, Monolith |
| Deployment section | Always |

Do not ask about components that are not part of the chosen shape.

---

## Phase 2: Derive from Vision

Using the **system shape** from Phase 1b, map each major concept in the vision to an architectural decision. Work through the vision top-to-bottom and identify only the concerns relevant to the chosen shape:

- **Actors / entities** → what components or services own them?
- **Flows** → how do they translate to API calls, messages, or actor interactions? *(backend/full-stack only)*
- **Storage needs** → what data must be persisted, and in what form? *(backend/full-stack/monolith)*
- **Real-time / async needs** → what communication patterns are required?
- **Concurrency / locking rules** → how are these enforced at the infrastructure level? *(backend/full-stack)*
- **Extensibility points** → how are configurable/swappable parts wired in?
- **UI communication** → how does the frontend talk to the backend? *(full-stack/monolith)*

For anything the vision does not decide (e.g. specific libraries, deployment topology), ask the user one question at a time. Do not ask multiple questions in a single message.

---

## Phase 3: First Draft

Write a complete `ARCHITECTURE.md` draft using only the sections that apply to the **system shape** defined in Phase 1b.

```
# Architecture

## Overview
<1–2 sentences: what the system is and what it's built with>

--- INCLUDE IF SHAPE = Backend | Full-stack ---
## API Layer
<how the system is exposed: REST/gRPC/GraphQL, contract-first or code-first, auth>
---

--- INCLUDE IF SHAPE = Backend | Full-stack | Monolith ---
## Backend
### Runtime & Startup
### Key Components / Services / Actors
<table: name | scope | responsibility>

## Data & Persistence
<databases, event stores, file storage, caching>

## <Domain-specific sections>
<one section per complex domain concern, e.g. Repository Lock, Memory Management, Task Execution>
---

## Multi-Tenancy (if applicable)
<how tenant isolation is enforced>

## Notifications (if applicable)
<notification hub pattern, delivery channels>

## AI Integration (if applicable)
<provider abstraction, config per tenant, default>

--- INCLUDE IF SHAPE = Frontend | Full-stack | Monolith ---
## Frontend
<SPA framework, communication protocol with backend, build toolchain>
---

## Deployment
<Docker Compose, Kubernetes, cloud targets, local dev setup>
```

Omit any section marked with INCLUDE IF when the system shape does not match. Do not add placeholder headings for absent components.

Present the draft inline. Do not write the file yet.

---

## Phase 4: Gap Review

After presenting the draft (or when enhancing an existing file), check for the following. Ask about any that are missing or ambiguous — one at a time:

**Consistency with vision:**
- Does every concept in the vision have a clear owner in the architecture?
- Are all failure/retry/stale behaviors from the vision reflected in the architecture?
- Are all extensibility points (configurable providers, notification channels) modelled?

**Technical decisions (backend / full-stack / monolith only):**
- Is it clear how entities are identified/addressed (IDs, shard keys)?
- Is the read vs. write path defined — when does something go through an actor vs. directly to the DB?
- Is the real-time communication pattern clear (WebSocket, gRPC streaming, polling)?
- Are background jobs or scheduled tasks identified (e.g. stale detection timers)?
- Is the auth flow described (token shape, claims, how tenantId is carried)?
- Is the plan/import pipeline described if the product ingests structured data?

**Technical decisions (frontend / full-stack / monolith only):**
- Is the UI framework and build toolchain specified?
- Is it clear how the frontend authenticates and calls the backend?
- Are real-time updates handled (polling, WebSocket, gRPC-Web, SSE)?

**Deployment & operations:**
- Is local development setup described?
- Is cluster discovery / service mesh covered for multi-node deployments? *(backend/full-stack)*
- Is configuration injection described (env vars, secrets, config maps)?

Only ask questions that apply to the established system shape — skip the rest.

If the user says "defer that", note it in the document as `— to be defined`.

---

## Phase 5: Iterate

Incorporate answers and present an updated draft. Repeat Phase 4 until the user confirms the architecture is complete.

Ask: *"Is there anything else missing, or shall I write the file?"*

---

## Phase 6: Write the File

Once the user confirms:

- **If enhancing an existing file** — apply only the changes needed. Preserve all existing content that is still accurate; update or extend sections that changed; add new sections for gaps that were filled.
- **If creating a new file** — write the full content.

Confirm when done: *"ARCHITECTURE.md updated. If you'd like to plan the implementation, run `/maggus-plan` and I'll break it down into tasks."*

---

## Style Guidelines

- Use **bold** for key terms and component names on first introduction
- Use tables for component inventories (name | scope | responsibility)
- Use bullet lists for properties and constraints — not prose paragraphs
- Prefer concrete names (`RepositoryActor`, `GridFS`, `Cluster Sharding`) over generic descriptions
- Explicitly state shard keys, default values, and configuration knobs
- Mark deferred decisions clearly rather than omitting them
- Keep domain-specific sections focused — one section per complex concern, not one mega-section
