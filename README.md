# Echo Doj0

## One-Line Pitch

Echo Doj0 is a privacy-first voice memory and AI reflection platform that turns personal voice entries into searchable, grounded memory with clear provenance.

## Demo

- Demo GIF/video: TODO - add a short walkthrough using synthetic demo data.
- Screenshots: TODO - add sanitized screenshots of recording, transcription review, reflection, retrieval, and provenance views.
- Live demo: TODO - add a link only if a safe public demo environment becomes available.

## Why This Exists

People capture thoughts, voice notes, and journal entries across scattered tools, but most systems stop at storage. They do not turn those moments into searchable, grounded, privacy-aware memory that can be reviewed, retrieved, and reflected on later.

Echo Doj0 is designed around the idea that personal memory tools should preserve original context, make AI output inspectable, and keep private source material separate from public showcase artifacts.

## What It Does

Echo Doj0 supports a voice-first memory loop:

- Record voice entries through a focused web interface.
- Preserve durable audio memory before downstream processing.
- Transcribe entries into reviewable text.
- Let users review and edit transcripts before relying on them as memory.
- Search and retrieve relevant past entries.
- Generate grounded AI reflections from user-authored material.
- Show receipts and provenance so reflections can be traced back to source entries.
- Support a companion/orb interface for recording, review, and reflection.
- Prepare for future emotional-intelligence practice loops built on trusted memory.

## Core Product Loop

**Record -> Transcribe -> Store -> Reflect -> Retrieve -> Grow**

- **Record:** Capture a voice entry in the browser.
- **Transcribe:** Convert the audio into editable text.
- **Store:** Preserve durable source artifacts and derived memory records.
- **Reflect:** Use AI to summarize, enrich, and respond with grounded context.
- **Retrieve:** Search past entries and surface relevant memories with receipts.
- **Grow:** Turn recurring patterns into future practice loops and better self-understanding.

## Key Features

| Feature | What it shows | Why it matters |
|---|---|---|
| Voice capture | Browser-based recording flow | Demonstrates practical audio UX and media ingestion design |
| Transcription | Speech-to-text conversion for entries | Turns raw voice notes into reviewable, searchable memory |
| Durable storage | Source audio and memory artifacts are preserved | Keeps original material available for replay, verification, and future processing |
| Reflection | AI-generated responses grounded in user-authored context | Shows applied LLM integration beyond generic chat |
| Receipts/provenance | Reflections point back to source memories | Builds trust by making AI output inspectable |
| Retrieval | Search and contextual recall over past entries | Makes memory useful after the original capture moment |
| Observability | Pipeline behavior can be traced at a high level | Supports debugging, evaluation, latency visibility, and reliability work |
| Privacy-first design | Public artifacts are sanitized and private data stays private | Shows disciplined boundaries around sensitive personal material |
| Agent-assisted engineering process | Planning, implementation, and verification use Codex-assisted workflows | Demonstrates modern AI-assisted development without overstating autonomy |

## Architecture Overview

Echo Doj0 is a private product with a public showcase boundary. This repository is intended to explain the system shape without exposing private source code, exact schemas, service internals, credentials, or proprietary scoring logic.

At a high level:

- **Frontend:** A React-based interface handles voice capture, transcript review, search, reflection display, and the companion/orb experience.
- **Backend:** A Node service coordinates API requests, audio ingestion, transcription jobs, AI enrichment, retrieval, and memory lifecycle updates.
- **Data layer:** Application records track entries, transcript state, metadata, retrieval context, and provenance relationships.
- **Storage:** S3-compatible object storage preserves durable audio and related artifacts.
- **AI pipeline:** Transcription and LLM-powered enrichment convert voice entries into searchable, reviewable, grounded reflections.
- **Retrieval/provenance:** Source entries and derived memory chunks are kept conceptually separate so reflections can cite the memories that informed them.
- **Observability:** AI and pipeline behavior are monitored with privacy-aware traces and operational metadata rather than published private content.

## Tech Stack

| Layer | Tools | Purpose |
|---|---|---|
| Frontend | React, Vite | Build the interactive recording, review, search, and reflection UI |
| Styling | Tailwind | Support a focused, responsive interface |
| Backend | Node, Express | Handle API orchestration, ingestion, and service coordination |
| API | GraphQL, Apollo | Manage structured client/server data flow |
| Data | MongoDB, Mongoose | Store memory records, processing state, and app metadata |
| Storage | S3-compatible object storage | Preserve durable audio and related artifacts |
| AI | OpenAI Whisper, LLM APIs | Transcribe audio and generate grounded reflections |
| Observability | Langfuse or AI observability tooling | Trace AI pipeline behavior, latency, and errors without publishing private content |
| Workflow | CI, Codex-assisted development | Support repeatable checks, docs-first planning, and reviewable changes |

## AI / Agentic Workflow

Echo Doj0 uses AI as part of a bounded product workflow rather than as an unreviewed autonomous system.

- **Transcription:** Voice entries are converted into text that can be reviewed before becoming trusted memory.
- **Enrichment:** AI can identify themes, summarize context, and prepare entries for retrieval.
- **Reflection:** Responses are generated from relevant user-authored context instead of unsupported freeform claims.
- **Receipts:** Reflections are designed to show the source memories that informed them.
- **Retrieval context:** The system assembles scoped context from stored memory before generating responses.
- **Observability:** Pipeline stages are monitored so failures, latency, and behavior can be inspected.
- **Human review and privacy boundaries:** Users remain responsible for reviewing memory content, and private prompts, raw data, and sensitive internals are not included in public artifacts.

## Privacy and Security Posture

This public repository is planned as a showcase, not the private source repository.

- No private source code is included.
- Demo data is synthetic.
- Secrets and user data are excluded.
- Raw audio and transcripts are not published.
- Architecture diagrams are intentionally sanitized.
- Public materials are designed to demonstrate product thinking, engineering process, and system architecture without exposing private implementation details.

## What Is Private / Redacted

- Private AE source code.
- Environment variables.
- Private prompts.
- Raw transcripts and audio.
- Proprietary scoring formulas.
- Private user data.
- Production URLs and credentials.
- Sensitive implementation internals.

## Engineering Process

Echo Doj0 is developed with a docs-first and privacy-aware engineering workflow:

- Product and architecture work is planned in markdown before implementation.
- Branch discipline keeps changes scoped and reviewable.
- CI and local checks are used to protect core behavior.
- Codex-assisted workflows help with planning, refactoring, documentation, and verification.
- Public artifacts receive privacy review before publishing.
- The private product repository remains the source of truth for implementation, while the public showcase repository communicates the product, architecture, and engineering signal safely.

## Current Status

This is a portfolio/showcase draft for a private in-development product. The showcase repo is intended to demonstrate architecture, product thinking, and engineering process without releasing proprietary source.

## Roadmap

- Public README.
- Architecture diagrams.
- Screenshots/GIF.
- 60-second demo.
- Portfolio/LinkedIn integration.
- Optional case study.

## About the Builder

William is a former high school English teacher transitioning into AI and full-stack engineering. His work combines communication, product thinking, applied AI systems, and full-stack architecture, with particular attention to privacy, user trust, and making complex technical systems understandable.

## Contact

- Portfolio: TODO
- LinkedIn: TODO
- GitHub: TODO
- Email: TODO

## Good Enough Marker

This README is ready when an employer can understand the product, technical architecture, privacy boundary, and William's engineering signal in under five minutes.
