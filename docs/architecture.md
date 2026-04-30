# Echo Doj0 Architecture

Echo Doj0 is a privacy-first voice memory and AI reflection platform.

This public architecture page explains the system shape at a high level without exposing private source code, production secrets, exact schemas, internal prompts, private user data, or proprietary scoring logic.

The private product repository remains closed-source. This showcase repository exists to demonstrate product architecture, engineering judgment, and AI/full-stack implementation strategy.

---

## 1. System Overview

Echo Doj0 is built around a simple but powerful product loop:

```text
Record -> Transcribe -> Store -> Reflect -> Retrieve -> Grow
```

At its core, the product is a voice-first memory archive. A user records a personal audio entry, the system preserves the durable source artifact, transcribes it, enriches it with AI, and later retrieves relevant memories with provenance.

The major system layers are:

| Layer | Responsibility |
|---|---|
| Frontend | Recording UI, transcript review, search, reflection display, companion/orb interaction |
| Backend API | Audio ingestion, GraphQL data access, orchestration, lifecycle updates |
| Database | Durable application records, entry metadata, transcript state, memory artifacts |
| Object Storage | Durable audio blob storage |
| AI Pipeline | Transcription, enrichment, reflection, retrieval context assembly |
| Retrieval Layer | Search, memory recall, grounding, provenance |
| Observability | AI/pipeline tracing, latency/error visibility, debugging metadata |
| Public Showcase Boundary | Sanitized diagrams, screenshots, docs, and demo assets only |

---

## 2. High-Level Architecture

```text
User
  |
  v
Browser / React UI
  |
  | 1. Record audio
  | 2. Upload .webm blob
  v
Backend API
  |
  |--> Object Storage
  |      - durable audio artifact
  |
  |--> Database
  |      - entry record
  |      - transcript state
  |      - pipeline status
  |
  |--> AI Pipeline
         - transcription
         - enrichment
         - reflection
         - retrieval context
         - provenance / receipts
```

The architecture favors durable source material first. Audio is treated as the primary memory artifact. Transcription, enrichment, and reflection are valuable secondary layers, but they must never block the user from saving the original memory.

---

## 3. Voice Ingestion Pipeline

The voice pipeline is the core MVP path.

### Step 1: Capture

The frontend records browser audio through a focused voice-recording interface.

Public-safe summary:

```text
Browser microphone
  -> audio blob
  -> upload request
```

### Step 2: Secure the Source Artifact

The backend receives the audio payload and stores the durable audio artifact in object storage.

Public-safe summary:

```text
Audio upload
  -> object storage
  -> entry record created
  -> entry ID returned to client
```

### Step 3: Mercy Mode

Echo Doj0 uses a “durable memory first” UX rule.

The user should not be trapped waiting for AI transcription or reflection. Once the durable audio memory exists, the interface can let the user continue while downstream enrichment finishes.

```text
Audio secured first.
AI enrichment can finish after.
User remains unblocked.
```

### Step 4: Targeted State Sync

The frontend checks the state of the specific newly-created entry rather than polling broad entry lists.

This avoids stale UI states and keeps the recording flow predictable.

---

## 4. AI / Reflection Pipeline

Echo Doj0 uses AI inside a bounded product workflow.

The AI system should not behave like an ungrounded chatbot. It should work from user-authored material and return reflections that can be inspected.

```text
Durable audio
  -> transcription
  -> transcript review
  -> enrichment
  -> reflection
  -> receipts / provenance
```

### Pipeline Stages

| Stage | Purpose | Public-Safe Explanation |
|---|---|---|
| Transcription | Convert voice to text | Turns audio into reviewable and searchable memory |
| Enrichment | Extract useful structure | Identifies themes, emotional tone, and useful metadata |
| Reflection | Generate response | Produces grounded AI reflection from user-authored context |
| Retrieval Context | Find relevant memory | Pulls prior related memories when useful |
| Receipts | Ground the output | Shows which memories or entries informed the response |
| Observability | Debug the pipeline | Tracks timing, errors, and behavior without exposing private content |

---

## 5. Memory, Retrieval, and Provenance

Echo Doj0 is designed around the idea that memory must be grounded.

The system should not imply that it remembers something unless it can point back to the source material that supports that claim.

### Memory Model

At a high level, memory is split into:

| Concept | Meaning |
|---|---|
| Source artifact | Original user-created material, such as audio or an entry |
| Derived artifact | Transcript, enrichment, summary, or searchable chunk |
| Retrieval result | A matched memory surfaced for a specific context |
| Receipt | A user-facing explanation of what source informed the output |

### Retrieval Principles

The retrieval layer should be:

- user-scoped
- provenance-aware
- score-gated
- explainable
- safe to degrade when retrieval is unavailable
- careful not to hallucinate continuity

### Provenance Principle

A reflection should be able to answer:

```text
What source material informed this?
Why was this memory retrieved?
How confident is the system?
Can the user inspect or reject the interpretation?
```

This is the difference between generic AI output and trustworthy AI memory.

---

## 6. Privacy Boundary

This public showcase repository intentionally does **not** include private implementation code.

### Public-Safe

This repo may include:

- high-level architecture diagrams
- product screenshots using synthetic data
- demo GIFs using synthetic data
- public README copy
- privacy/security posture
- product overview
- case-study writing
- sanitized engineering process notes

### Private / Redacted

This repo must not include:

- private AE source code
- environment variables
- service credentials
- production URLs
- private prompts
- raw audio
- raw transcripts
- private user data
- database dumps
- vector store exports
- trace payloads
- proprietary scoring formulas
- sensitive internal implementation details

The public repo is a case study and showcase, not a source release.

---

## 7. Tech Stack

| Layer | Tools | Purpose |
|---|---|---|
| Frontend | React, Vite | Interactive recording, transcript review, search, reflection UI |
| Styling | Tailwind CSS | Responsive, focused, product-specific interface styling |
| API | GraphQL / Apollo | Structured client-server data access |
| Backend | Node.js, Express | API orchestration, audio ingestion, service coordination |
| Database | MongoDB, Mongoose | Durable app records, entry state, memory metadata |
| Object Storage | S3-compatible storage | Durable audio artifact storage |
| AI | OpenAI Whisper / LLM APIs | Transcription, enrichment, reflection |
| Observability | Langfuse or similar AI telemetry | Trace AI pipeline behavior and debugging metadata |
| Workflow | Git, GitHub CLI, CI, Codex-assisted development | Scoped implementation, verification, and review discipline |

---

## 8. Diagram Plan

The public showcase should eventually include three core diagrams.

### Diagram 1: Voice Ingestion Pipeline

Should show:

```text
Browser Recorder
  -> Upload API
  -> Object Storage
  -> Entry Record
  -> Transcript State
  -> UI Update
```

Purpose:

Show that the product handles real media ingestion, durable storage, and asynchronous AI processing.

### Diagram 2: AI Orchestration Pipeline

Should show:

```text
Entry
  -> Transcription
  -> Enrichment
  -> Reflection
  -> Retrieval Context
  -> Receipts
  -> Observability
```

Purpose:

Show applied AI orchestration without overclaiming autonomous agents.

### Diagram 3: Memory / Retrieval / Provenance

Should show:

```text
Source Artifacts
  -> Derived Searchable Memory
  -> Scoped Retrieval
  -> Grounded Reflection
  -> User-Visible Receipts
```

Purpose:

Show that Echo Doj0 treats memory as evidence-backed, not magical.

---

## 9. Engineering Principles

Echo Doj0 is designed around a few technical and product guardrails.

### Durable Memory First

The user’s original audio memory matters more than any downstream AI output.

If AI fails, the entry should still exist.

### AI as Enrichment, Not Authority

AI reflections are secondary interpretations. They should be inspectable and grounded.

### Provenance Over Vibes

The system should be able to show why it surfaced a memory or made a reflective claim.

### Privacy by Default

Public artifacts must use synthetic data and sanitized diagrams.

### Explicit State

The UI should distinguish between:

```text
uploaded
transcribing
ready
partial
failed
```

A missing transcript is not the same thing as a failed memory.

### Public / Private Separation

The private AE repo is the source of truth.
The public showcase repo is the proof-of-work artifact.

---

## 10. What This Architecture Demonstrates

For employers, this project demonstrates:

| Area | Signal |
|---|---|
| Full-stack engineering | React frontend, Node backend, GraphQL API, MongoDB data modeling |
| Applied AI | Transcription, enrichment, reflection, retrieval context |
| Media handling | Browser recording, binary upload, durable audio storage |
| Product thinking | Voice-first memory loop, Mercy Mode, user trust boundaries |
| AI safety / trust | Receipts, provenance, privacy-aware design |
| Systems thinking | Async pipeline, source/derived artifact separation, lifecycle state |
| Developer workflow | Git discipline, CI, scoped branches, Codex-assisted iteration |
| Communication | Clear public documentation and employer-facing architecture explanation |

---

## 11. Current Public Showcase Status

This public repository is a showcase scaffold for a private in-development product.

Current public artifacts:

- README draft
- architecture overview
- privacy/security page placeholder
- product overview placeholder
- AI system overview placeholder
- development process placeholder
- roadmap placeholder
- case-study placeholders
- screenshot / diagram / demo asset folders

Next expected additions:

1. Sanitized architecture diagrams
2. Synthetic-data screenshots
3. 60-second demo GIF/video
4. Portfolio link
5. LinkedIn project entry
6. Optional case study

---

## 12. Good Enough Marker

This architecture page is ready when an employer can understand the system shape, technical credibility, privacy boundary, and AI/full-stack engineering signal without seeing the private source code.
