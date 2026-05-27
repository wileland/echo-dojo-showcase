# Echo Doj0

## One-Line Pitch

Echo Doj0 is a private-first voice memory archive and AI reflection platform — built around the idea that every AI insight should trace back to the user's own words, and that the system should reflect what it can prove.

## Live App

- **Live:** [ashlight-frontend.onrender.com](https://ashlight-frontend.onrender.com)
- **Portfolio:** [williamhaynesportfolio.com](https://williamhaynesportfolio.com)
- Screenshots: coming — sanitized walkthrough of the core reflection flow
- Demo GIF: coming — 60-second record → reflect → receipt sequence

## Why This Exists

Most tools that handle personal voice and journal entries stop at storage. They do not turn those moments into searchable, grounded, privacy-aware memory that can be reviewed, retrieved, and reflected on with evidence.

Echo Doj0 is built around one constraint: **the system should not guess**. It should remember, retrieve, cite, and reflect from the user's own words. Every AI claim carries a receipt. The system earns what it says.

The product loop is:

**Record → Reflect → Recognize → Grow**

## What It Does

- Record voice entries through a focused web interface.
- Preserve durable audio memory before any downstream AI processing.
- Transcribe entries into reviewable text.
- Generate grounded AI reflections from user-authored material only.
- Show receipts so every reflection traces back to its source entry.
- Search and retrieve relevant past entries with provenance.
- Support a companion interface that reflects without diagnosing.
- Build toward emotional-intelligence practice loops grounded in the archive.

## Core Principles (Enforced in Code)

**Provenance is Sacred** — every receipt hashes to a real source entry. The receipt is the anti-bullshit architecture.

**Mirror, not Oracle** — the companion reflects what it can prove. Runtime guardrails screen generated responses for overclaiming before delivery. The mirror that guesses is a liar.

**Receipt before Reward** — XP, progression, and gamification follow the receipt. Never precede it.

**The archive is the load-bearing stone** — the game serves the archive. Never the reverse.

## Key Features

| Feature | What it shows | Why it matters |
|---|---|---|
| Voice capture | Browser-based recording flow | Practical audio UX and media ingestion design |
| Transcription | Whisper speech-to-text | Turns raw voice notes into reviewable, searchable memory |
| Durable storage | Source audio and memory artifacts preserved | Original material available for replay and future processing |
| Reflection | AI responses grounded in user-authored context only | Applied LLM integration beyond generic chat |
| Receipts/provenance | Reflections trace back to source memories | Trust through inspectability |
| Retrieval | Semantic search with Atlas Vector Search | Makes memory useful after the original capture moment |
| Observability | Langfuse pipeline tracing | Debugging, evaluation, and latency visibility without PII |
| Privacy-first design | Public artifacts sanitized, private data stays private | Disciplined boundaries around sensitive personal material |
| Agent-assisted engineering | Codex-assisted planning, implementation, verification | Modern AI-assisted development without overstating autonomy |

## Architecture Overview

This repository is the public showcase boundary. It explains the system shape without exposing private source code, schemas, service internals, credentials, or proprietary scoring logic.

- **Frontend:** React + Vite — voice capture, transcript review, search, reflection display, companion/orb experience.
- **Backend:** Node + Express — API orchestration, audio ingestion, transcription jobs, AI enrichment, retrieval, memory lifecycle.
- **API:** GraphQL + Apollo — structured client/server data flow with a dual typedef tree for CI enforcement.
- **Data:** MongoDB Atlas — durable entry records, vector memory chunks, pipeline state, provenance metadata.
- **Storage:** AWS S3 — durable audio blob storage. Key-based storage separates stable keys from ephemeral signed URLs.
- **AI pipeline:** OpenAI Whisper (transcription) + GPT-4o-mini (reflection) + Anthropic claude-3-5-haiku (cognitiveSignals derivation). Non-interchangeable clients with separate billing and separate responsibilities.
- **Retrieval:** MongoDB Atlas Vector Search — `text-embedding-3-small`, 1536 dims, cosine similarity, score-gated at 0.72 hallucination firewall.
- **Queue:** BullMQ + Redis — async pipeline orchestration for audio ingestion and enrichment workers.
- **Observability:** Langfuse — two-layer span pattern, no PII in any span payload, ever.

## Tech Stack

| Layer | Tools |
|---|---|
| Frontend | React, Vite, Tailwind CSS, Framer Motion |
| Backend | Node.js, Express |
| API | GraphQL, Apollo Client + Server |
| Database | MongoDB Atlas (vector search + document store) |
| Storage | AWS S3 |
| Auth | Supabase |
| AI — Reflection | OpenAI GPT-4o-mini, Whisper |
| AI — Enrichment | Anthropic claude-3-5-haiku |
| Queue | BullMQ, Redis |
| Observability | Langfuse |
| Deployment | Render (three services: backend, worker, frontend) |
| Workflow | Claude Code, Codex CLI, GitHub PR discipline |

## AI / Agentic Workflow

Echo Doj0 uses AI inside a bounded, receipt-governed product workflow.

- **Transcription:** Voice entries converted to reviewable text before becoming trusted memory.
- **Enrichment:** Anthropic-powered cognitiveSignals derivation — themes, patterns, behavioral signals.
- **Reflection:** Generated from retrieved user-authored context only. Not from generic prompts.
- **Receipts:** Every reflection cites the source entries that grounded it. Persisted at write time, not re-queried at read time.
- **Retrieval governance:** Hybrid retrieval with score gating. Anti-doom-loop exclusion via `seenChunkIds`. Hallucination firewall at 0.72 — not a tuning detail.
- **Observability:** Pipeline stages traced for latency, errors, and token usage. No raw user content in any trace payload.

## Privacy and Security Posture

This repository is a showcase, not the private source repository.

- No private source code is included.
- Demo data is synthetic.
- Secrets and user data are excluded.
- Raw audio and transcripts are not published.
- Architecture descriptions are intentionally sanitized.
- The `$vectorSearch.filter.userId` is a sovereignty boundary, not a tenancy convenience. A user recording their inner monologue at 2am is trusting this system with the most private thing they produce.

## Engineering Process

- Architecture and product work planned in markdown before implementation.
- Branch discipline: all PRs target `develop`, promoted to `main` via squash merge with parity verification.
- CI enforces dual typedef trees, test floor, lint, and build on every PR.
- Codex CLI and Claude Code used for implementation; adversarial council review before any agent fires on production code.
- 1,500+ pull requests. Solo. No team. No funding.
- The quality bar stayed high because regressions were treated as product problems, not chores.

## Current Status

Echo Doj0 is an active in-development product with a demo-viable core:

- Session Zero threshold ritual live
- Voice → transcript → Atlas storage pipeline live
- AI reflection path verified (grounded, receipt-backed)
- Entry/reflection surface polished for demo
- Pending Trial / Crucible practice loop in progress
- Vibe Gravity measurement system designed, write path in progress

This is not a tutorial project. It is a working system with real decisions at every layer.

## About the Builder

**William L. Haynes** — founding engineer, former 10-year English teacher, novelist.

A decade in ESOL and English classrooms sharpened the instincts that make a founding engineer dangerous: the ability to break complex systems into legible steps, document clearly under pressure, and understand what a user actually needs versus what they say they want.

Echo Doj0 is what happens when a novelist's ear and an engineer's rigor work on the same problem. Built solo. Architecturally complete. Ready for founding engineer, full-stack AI, and mission-driven product roles.

> *"Same rigor as enterprise agentic RAG. Higher intimacy. More human stakes."*

## Contact

- **Portfolio:** [williamhaynesportfolio.com](https://williamhaynesportfolio.com)
- **LinkedIn:** [linkedin.com/in/williamhaynesxp](https://www.linkedin.com/in/williamhaynesxp/)
- **GitHub:** [github.com/wileland](https://github.com/wileland)
- **Phone:** (210) 775-8143
- **Email:** wileland7@gmail.com
