# Changelog

All notable changes to the Deep-Reading Assistant are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Pre-2.0 entries are reconstructed retroactively from git history and therefore list the significant changes per era rather than a per-commit log.

---

## [2.0.0] — Full-stack multi-agent revamp

The first release under the new architecture. The original synchronous `LibrarianAgentsTeam` prototype has been retired; the project is now a FastAPI + Next.js application driven by an async orchestrator-workers agent loop.

### Added

- **Splash → Home → Session flow** — a three-page experience with a landing splash (`/`), a greeting/draft-upload home (`/home`), and per-session chat pages (`/sessions/[id]`).
- **Persistent chat sidebar** — list, pin, rename, and delete sessions. Auto-generated titles from the first user message.
- **Edit and retry** — edit a prior user message or retry any assistant reply; the backend truncates history from the pivot message onward via `DELETE /api/sessions/{id}/messages/after/{message_id}`.
- **Thinking panel** — Lead and subagent reasoning stream as `thinking_delta` events, rendered as a collapsible "Thinking…" block separate from the final answer.
- **Stable `final_message` contract** — the user-facing answer is delivered as a single SSE event, cleanly decoupled from the reasoning stream.
- **Artifact preview canvas** — artifacts slide in as a flex-row sibling of the chat (not an overlay) with Markdown, HTML, CSV, and plain-text rendering and a download button.
- **Persistent agent trace** — `agent_spawned`, `tool_use`, `artifact_written`, `agent_done`, and `compaction_done` events are persisted to a new `trace_events` table and replayed on session load so history survives reloads.
- **Draft sessions** — drop files on the home screen before composing a prompt; a session is created lazily on first upload.
- **Drag-and-drop everywhere** — both the home screen and session pages accept dropped files.
- **Session pinning** — `sessions.pinned` column + `PATCH /api/sessions/{id}` support pinning from the sidebar.
- **FastAPI backend** — async routes for sessions, documents, messages, SSE stream, context meter, compaction trigger, trace replay, artifact fetch, and chunk fetch.
- **SQLite schema** — `sessions`, `messages`, `documents`, `chunks`, `chunks_fts` (FTS5), `definitions`, `cross_refs`, `artifacts`, `agent_runs`, `trace_events`.
- **Async multi-agent orchestration** — `LeadOrchestrator` with `search_document`, `read_document_chunk`, `resolve_reference`, `lookup_definition`, `spawn_subagent`, `write_artifact`, and `finalize` tools. Subagents run in parallel via `asyncio.gather`.
- **Prompt caching** — ephemeral cache control on Lead and SubAgent system prompts (1 h TTL).
- **Citation enforcement** — subagent outputs are regex-scanned for UUID citations; uncited results are flagged and returned to the Lead with a warning note.
- **Automatic compaction** — at 85% of the 200K token window, the Haiku compactor summarises earlier turns into a structured memory so the session continues without interruption.
- **FTS5 query safety** — `_fts5_safe` tokenises and quotes LLM-generated queries so operators cannot reach the parser.
- **Definition and cross-reference extractors** — run as background tasks on upload to populate `definitions` and `cross_refs`.
- **Typed frontend API layer** — `api.ts` and `sse.ts` with full TypeScript types for every REST response and SSE event.
- **Direct-to-backend SSE** — the SSE client bypasses the Next.js proxy (which buffers streamed responses) and connects directly to port 8000. Override with `NEXT_PUBLIC_SSE_BASE`.
- **Source drawer with filename context** — `GET /api/chunks/{id}` returns the user-facing document filename so citations render meaningful labels instead of opaque hashes.
- **Finalize safety net** — if the Lead wrote an artifact but returned an empty `result`, a fallback recap pointing at the artifact name is auto-generated so the UI never shows a blank message.
- **Async CLI** — `cli.py` now drives the same `run_lead` entry point as the web app, with `--interactive`, `--audience`, `--verbose`, and `-o` flags.
- **technical_docs.md** — comprehensive developer reference covering the API, schema, orchestration, SSE protocol, and extension points.

### Changed

- **Version bump** — `frontend/package.json` `"version": "0.1.0"` → `"2.0.0"`, reflecting the scale of the rewrite. Backend FastAPI app version is `1.0.0`.
- **Models** — dev default is now `claude-haiku-4-5-20251001` across Lead, SubAgent, and Compactor for cost control during local work. For production, swap the Lead to `claude-opus-4-6` in [backend/orchestrator/lead.py](backend/orchestrator/lead.py) or set `ANTHROPIC_MODEL`.
- **Chunking** — default chunk size lifted to 8,000 characters with page/chapter/section metadata surfaced into the `chunks` table.
- **Document display name** — `documents.original_filename` is the user-visible string; the OS temp path used during ingest never reaches the UI.
- **Session detail response** — now returns `{ session, messages, documents, artifacts }` in a single round-trip.
- **Auto-titling** — session titles are derived from the first line of the first user message (max 60 chars).
- **Home → Session handoff** — `/sessions/<id>?prompt=…&audience=…` passes the initial prompt so the session page can render the user message and open SSE immediately.

### Removed

- **`librarian_agents_team.py`** — the original synchronous prototype.
- **`advanced_examples.py`** — scenario demos that depended on the sync prototype.
- **`test_example.py`** — end-to-end workflow demo that depended on the sync prototype.
- **Earlier README version-history section** — superseded by this file.

### Fixed

- **Session retrieval** — replaced `execute_fetchone` with explicit `execute` + `fetchone` to align with `aiosqlite`'s API.
- **Database initialisation** — `_init_db` guards against re-entry via a module-level `_db_initialised` flag; the `get_db()` async context manager initialises lazily on first use.
- **Live context meter** — seeded from `GET /api/sessions/{id}/context` on session load so the bar is never stuck at 0%.
- **EventBus replacement** — `ensure_live` replaces a closed bus so a new run doesn't publish into a dead queue.

### Security

- **FTS5 injection hardening** — see `_fts5_safe` above.
- **CORS allowlist** — `http://localhost:3000` and `http://127.0.0.1:3000` only; tighten or extend deliberately before deploying.

---

## [1.x] — Synchronous prototype era *(reconstructed)*

Prior to the 2.0 revamp, the project shipped as a single-file synchronous agent system built on the Anthropic SDK. There were no numbered releases; this entry captures the shape of the code at the point the rewrite began.

### Feature set

- `LibrarianAgentsTeam` — a synchronous orchestrator + workers implementation in [librarian_agents_team.py](librarian_agents_team.py) (removed in 2.0).
- Multi-format document loader (`DocumentLoader`) and chunker (`DocumentChunker`) — both retained in 2.0.
- Prompt caching on librarian agents ([`3b52527a added prompt caching to librarian agents team`](https://github.com/wei-kiat-tan/GenAI/commits/main)).
- Scenario demos (`advanced_examples.py`) and an end-to-end smoke test (`test_example.py`) — both removed in 2.0.
- Project summary documentation in `PROJECT_SUMMARY.md` (later consolidated into README).
- README and ARCHITECTURE documentation describing the sync prototype.

### Notable commits during this era

- `ff8f39e9` — initial `LibrarianAgentsTeam` implementation.
- `3b52527a` — prompt caching added to the librarians.
- `694dbd18` — ReadME overhaul.
- `81a2bb0a` — version history section added to README.
- `f6eb556d` / `80d69e2c` — README updates.
- `d5ed05bd` — removed the inline `__main__` demo from `document_chunker.py`.

---

## [0.x] — Initial sync prototype *(reconstructed)*

The project started as a small experiment around the Anthropic API with ad-hoc scripts.

### Notable commits

- `e2744efa`, `272bbddc` — initial file uploads.
- `abac4f35` — Anthropic model and `max_tokens` adjustments.
- `d13bb627` / `47e3f06a` — early test fixtures (`testing2`).
- `282e149a` — rename `librarian-multi-agents` to `spare`.

---

## Conventions for future entries

- Group changes under **Added**, **Changed**, **Deprecated**, **Removed**, **Fixed**, **Security**.
- Link to relevant source files and commits where it clarifies scope.
- Date each release (`YYYY-MM-DD`) when cut. Unreleased work sits under a `[Unreleased]` heading at the top.
- Bump the frontend `package.json` version and (if the API contract changes) the FastAPI `app.version` in lockstep with each release.
