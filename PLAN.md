# Multi-Agent Document Processing System — Full-Stack Plan

## Context

The repo currently contains a CLI-only, synchronous Python prototype ([librarian_agents_team.py](librarian_agents_team.py)) that orchestrates a Lead agent and three hardcoded SubAgents against Anthropic's Messages API. The goal is to evolve this into a production-grade **multi-agent system for large document processing** with:

- A **dynamic agent topology** (Lead + N SubAgents, not fixed at 3), built per Anthropic's recommended patterns.
- A **web frontend** where users upload documents, issue prompts, and watch the agent team work in real time.
- A **backend API** that streams progress, persists sessions, and scales to large documents (50–5000+ pages).

The existing document loaders and chunker are solid and will be reused. The orchestration layer needs a rewrite — current code is sequential, has only 3 fixed subagents, and has no web surface.

**Confirmed design decisions:**
- Subagents spawned **dynamically via tool use** (orchestrator-workers pattern).
- Frontend: **Next.js 15 + shadcn/ui**.
- Persistence: **SQLite + local filesystem** (swappable later).
- **CLI retained**, repointed to the new async engine.

---

## Architecture Overview

```
┌────────────────────────────────────────────────────────────┐
│  Frontend (Next.js 15 + React + Tailwind + shadcn/ui)      │
│  - Chat UI, file upload, live agent trace panel            │
│  - SSE stream for agent events                             │
└──────────────────────────────▲─────────────────────────────┘
                               │  HTTPS + SSE
┌──────────────────────────────▼─────────────────────────────┐
│  Backend (FastAPI + async)                                 │
│  - REST: /upload, /sessions, /messages                     │
│  - SSE: /sessions/{id}/stream                              │
│  - SQLite (sessions, messages, artifacts)                  │
└──────────────────────────────▲─────────────────────────────┘
                               │
┌──────────────────────────────▼─────────────────────────────┐
│  Agent Orchestration (anthropic SDK, async)                │
│  - LeadOrchestrator: planner + tool use                    │
│  - SubAgent pool: spawned per-task, parallel via           │
│    asyncio.gather                                          │
│  - Shared: DocumentStore, PromptCache, EventBus            │
└────────────────────────────────────────────────────────────┘
```

---

## Backend Design

### Agent orchestration (replaces [librarian_agents_team.py](librarian_agents_team.py))

Follow Anthropic's **orchestrator-workers pattern** (docs: "Building effective agents"):

- **LeadOrchestrator** runs an agent loop with **tool use**. Its tools:
  - `spawn_subagent(role, task, context_refs)` — dispatches a subagent; returns a handle.
  - `read_document_chunk(chunk_id)` — pulls from DocumentStore.
  - `search_document(query)` — keyword/semantic lookup over chunks.
  - `write_artifact(name, content)` — persists intermediate results.
  - `finalize(result)` — ends the session.
- **SubAgents** are **not hardcoded roles** — the Lead writes a role description per spawn (e.g., "extract financial tables from chunks 4–7", "summarize chapter 3"). Each subagent runs its own Messages API call with its own tools (`read_document_chunk`, `write_artifact`, `request_clarification`).
- **Parallelism**: `asyncio.gather` over spawned subagents. Lead waits, then synthesizes.
- **Context isolation**: each subagent receives only the chunks it needs — keeps context windows small and costs down.
- **Prompt caching**: cache document chunks (1h TTL, `cache_control: ephemeral`) so repeated subagent calls over the same document are cheap. Cache per-subagent system prompts too.
- **Streaming**: `client.messages.stream()` on Lead and SubAgents; deltas pushed to the EventBus.
- **Subagent cloning**: the Lead can spawn N subagents with *identical* role + instructions but different tasks/chunk refs (e.g., 8 parallel summarizers, one per chunk range). This is the core win of dynamic topology over fixed roles.
- **Per-agent context management**: every subagent is a fresh Messages call with an isolated window — system prompt (cached) + only the chunk IDs it needs. No cross-subagent history. Subagents are ephemeral workers: start clean, return a result, exit. If a single subagent's task is still too large, the Lead splits further before spawning.
- **Lead-side compaction**: only the Lead has a growing conversation. When its context crosses ~70% of the 200K window, a compaction pass condenses older turns into a structured memory summary (decisions, artifacts produced, open threads) that replaces raw history. Artifacts live in the DB and are referenced by ID, not re-embedded. Auto-trigger at 85%; user can also trigger manually from the frontend.
- **Why prompt caching still matters even with chat history**: chat history keeps the Lead's memory; caching prevents re-paying for identical prefix tokens (document chunks + system prompts) across many subagent calls. 10 subagents over the same doc → 1× full-price + 9× ~10%-price reads. Cost and latency win; unrelated to memory.

### Tech stack

- **FastAPI** (async) + **uvicorn**
- **anthropic** Python SDK (async client)
- **SQLite** via `aiosqlite` — full persistence of chat history and run telemetry. Schema:
  - `sessions` (id, created_at, title)
  - `messages` (id, session_id, role, content, token_usage, created_at)
  - `documents` (id, session_id, filename, chunk_count)
  - `chunks` (id, document_id, index, content, metadata)
  - `artifacts` (id, session_id, name, content, mime_type)
  - `agent_runs` (id, session_id, parent_agent_id, role, status, tokens_in, tokens_out) — powers trace replay
- **Local filesystem** for uploaded documents
- **SSE** for streaming agent events to the browser (simpler than WebSockets for one-way push)
- **pydantic v2** for schemas
- Reuse: [document_loader.py](document_loader.py), [document_chunker.py](document_chunker.py)

### Key API endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/sessions` | Create a chat session |
| POST | `/api/sessions/{id}/documents` | Upload document (multipart); loader + chunker run; chunks stored |
| POST | `/api/sessions/{id}/messages` | Submit user prompt; kicks off agent run |
| GET | `/api/sessions/{id}/stream` | SSE stream: agent events |
| GET | `/api/sessions/{id}` | Session history + artifacts |
| GET | `/api/sessions/{id}/context` | Current Lead token usage + % of window |
| POST | `/api/sessions/{id}/compact` | Manually trigger Lead-context compaction |
| GET | `/api/artifacts/{id}` | Download generated artifact |

### Event schema (SSE)

```json
{ "type": "agent_spawned", "agent_id": "...", "role": "...", "parent": "lead" }
{ "type": "text_delta", "agent_id": "...", "delta": "..." }
{ "type": "tool_use", "agent_id": "...", "tool": "read_document_chunk", "input": {...} }
{ "type": "artifact_written", "artifact_id": "...", "name": "..." }
{ "type": "agent_done", "agent_id": "...", "summary": "..." }
{ "type": "run_complete", "final": "..." }
{ "type": "context_usage", "tokens": 68000, "window": 200000, "percent": 34 }
{ "type": "compaction_done", "before_tokens": 170000, "after_tokens": 42000 }
```

### Critical backend files (new)

- `backend/app.py` — FastAPI app + routes
- `backend/orchestrator/lead.py` — LeadOrchestrator with tool loop
- `backend/orchestrator/subagent.py` — SubAgent runner
- `backend/orchestrator/tools.py` — tool definitions + handlers
- `backend/orchestrator/event_bus.py` — in-process pub/sub for SSE
- `backend/orchestrator/compactor.py` — Lead-context compaction pass (condenses old turns into a memory summary)
- `backend/store/documents.py` — DocumentStore (wraps existing loader/chunker)
- `backend/store/sessions.py` — SQLite session persistence
- `backend/models.py` — pydantic schemas

Keep [document_loader.py](document_loader.py), [document_chunker.py](document_chunker.py) at repo root; import from `backend/`.

---

## Frontend Design

### Stack

- **Next.js 15** (App Router) + **React 19** + **TypeScript**
- **Tailwind CSS** + **shadcn/ui**
- **lucide-react** icons
- **react-markdown** + **rehype-highlight** for rendering agent output
- Native **EventSource** for SSE

### Layout

Three-pane layout:

```
┌────────────┬──────────────────────┬──────────────────┐
│ Sessions   │  Chat (main)         │  Agent Trace     │
│ sidebar    │  - messages          │  - live tree of  │
│ - new chat │  - file upload zone  │    agents &      │
│ - history  │  - input box         │    tool calls    │
│            │  - artifacts inline  │  - collapsible   │
└────────────┴──────────────────────┴──────────────────┘
```

### UX principles

- **Drag-and-drop upload** with progress, page count preview, per-file chunk stats.
- **Streaming first**: every agent token streams into the chat; the trace panel updates live as subagents spawn.
- **Transparency without noise**: trace panel is collapsible; default shows a compact "3 agents working…" indicator with expandable detail.
- **Artifacts**: tables, summaries, and generated files render inline with download buttons; long outputs open in a "view full" modal.
- **Interruptibility**: a Stop button cancels the run (backend sends `AbortSignal` to the Anthropic client).
- **Resume**: sessions persist — reload the page, pick up the history.
- **Context meter**: header strip shows live Lead-context usage — `Context: 34% (68K / 200K)`. Color-coded (green < 70%, amber 70–90%, red > 90%). A **Compact** button sits next to it — disabled below 30%, one-click above. Auto-compaction at 85% surfaces a toast with a "View summary" link.

### Critical frontend files (new)

- `frontend/app/layout.tsx` — root layout + theme
- `frontend/app/sessions/[id]/page.tsx` — main chat view
- `frontend/components/ChatPane.tsx`
- `frontend/components/AgentTrace.tsx` — renders SSE event tree
- `frontend/components/UploadZone.tsx`
- `frontend/components/ArtifactCard.tsx`
- `frontend/components/ContextMeter.tsx` — live token-usage bar + Compact button
- `frontend/lib/sse.ts` — typed EventSource wrapper
- `frontend/lib/api.ts` — fetch helpers

---

## Migration of existing code

| Existing file | Action |
|---|---|
| [document_loader.py](document_loader.py) | **Keep**, import from backend |
| [document_chunker.py](document_chunker.py) | **Keep**, import from backend |
| [librarian_agents_team.py](librarian_agents_team.py) | **Replace** with async orchestrator; keep as reference |
| [cli.py](cli.py) | **Keep**, repoint to new async orchestrator |
| [advanced_examples.py](advanced_examples.py), [test_example.py](test_example.py) | Repurpose as integration tests against the new backend |

---

## Implementation phases

1. **Backend scaffold** — FastAPI app, SQLite schema, document upload + chunk pipeline wired to existing loader/chunker. *Deliverable: POST a PDF, GET back chunks.*
2. **Async orchestrator** — LeadOrchestrator with tool use + dynamic SubAgent spawn via `asyncio.gather`. Prompt caching on chunks. Streaming. *Deliverable: end-to-end CLI run against the new async engine.*
3. **SSE event bus** — wire agent events to `/stream`. *Deliverable: `curl` the stream and see live events.*
4. **Frontend scaffold** — Next.js app, three-pane layout, session list, upload zone (no streaming yet).
5. **Live chat + trace** — SSE consumer, streaming renderer, agent trace tree.
6. **Artifacts + polish** — inline artifact cards, download, stop/resume, error states, loading skeletons.
7. **Hardening** — auth (simple API key to start), rate limits, error surfaces, token-usage display, tests.

---

## Verification

- **Unit**: chunker + loader tests (already exist); add orchestrator tool-handler tests with a mocked Anthropic client.
- **Integration**: run a 200-page PDF through the full stack; assert subagent spawn, parallel execution, final artifact.
- **Manual E2E**: upload a real document in the browser, issue a complex prompt ("extract all tables and summarize chapter 4"), watch the trace panel, verify artifacts download.
- **Load**: run two long documents concurrently in separate sessions; confirm no cross-session leakage and SSE streams stay distinct.
- **Cache hit rate**: log `cache_read_input_tokens` — expect high hit rates on repeated subagent calls over the same document.
