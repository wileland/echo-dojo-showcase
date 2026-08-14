# Echo Doj0

**A voice-first memory archive with an AI companion that can only say what it can prove.**

- **Live app:** [ashlight-frontend.onrender.com](https://ashlight-frontend.onrender.com)
- **Portfolio:** [williamhaynesportfolio.com](https://williamhaynesportfolio.com)
- **Deeper technical companion:** [`docs/architecture.md`](docs/architecture.md)

This is the public showcase repository. The product source stays private; what is published here is architecture and engineering judgment.

---

## What It Is

You speak into Echo Doj0. It preserves the recording, transcribes it, turns it into retrievable memory, and later reflects your own words back to you — with receipts showing exactly which past entries grounded what it said.

The product constraint is a single sentence: **the system does not guess.** Every reflective claim is either backed by retrieved, integrity-verified source material, or it is honestly labeled as not being backed by any.

That constraint is what makes the engineering interesting.

---

## Why This Is Technically Hard

A journal app is easy. A journal app whose AI is trustworthy is not.

- **The source material is irreplaceable.** A voice memory recorded at 2am cannot be regenerated. Every downstream stage — transcription, enrichment, reflection, indexing — must be allowed to fail without ever costing the user the original.
- **The valuable work is asynchronous.** Transcription and generation take seconds to minutes. The user cannot be held hostage to them, which means the system needs durable multi-stage lifecycle state rather than a request/response illusion.
- **Retrieval is longitudinal and adversarial-by-accident.** Memory is retrieved across months of a person's own writing. Raw nearest-neighbor search over that corpus returns repetitive, stale, or semantically-close-but-wrong material — and retrieved user content is itself untrusted input on its way into a model prompt.
- **Grounding has to be provable, not asserted.** "Cite your sources" is a prompt instruction. A receipt that survives a hash check is an architecture.
- **The data is unusually intimate.** Which makes the privacy boundary — especially around telemetry, where the temptation to log the interesting payload is strongest — a design constraint rather than a compliance checkbox.
- **Nothing may leak across users.** Cross-user retrieval in a system holding this material is not a bug class. It is the bug class.

---

## Production AI Systems Work

The parts of this system a senior engineer would actually want to interrogate.

### Governed retrieval, not raw vector search

Retrieval is a policy pipeline, not a similarity call. A query runs against a deep candidate pool, results below a calibrated similarity floor are discarded, surviving candidates are adjusted by a gentle recency decay, and a diversity-aware re-ranking step selects the final set so the companion does not keep surfacing five near-duplicates of the same memory.

On top of that sit governance filters: user-vetoed memories are excluded at the query filter itself; reference material ingested for retrieval only is structurally ineligible to become a user-facing receipt; and recently-surfaced chunks are excluded to break repetition loops — with that exclusion list explicitly bounded, because an unbounded exclusion set eventually breaks the query engine rather than the product.

### User scope as a security boundary

Retrieval is user-scoped inside the vector query filter, not filtered afterward in application code. A single enforcement point throws rather than executing when user scope is absent, so there is no code path that can accidentally issue a global query. This is treated as a sovereignty boundary, not a multi-tenancy convenience.

### Provenance that is verified, not claimed

Every candidate receipt is integrity-checked: the retrieved text is re-hashed and compared against the hash stored when the memory was written. A mismatch drops the chunk, and the warning is logged without any user content. Only verified receipts reach the model, and only verified receipts reach the user.

Receipts are then **persisted onto the assistant message at write time** — claim type, confidence, surfacing rule, and the cited excerpts — so the provenance drawer reads a durable snapshot of what actually grounded that response. It is never reconstructed by re-running a search later and hoping it lands in the same place.

### Bounded AI, subordinate to deterministic state

Context is assembled once into a sealed, frozen envelope and passed unchanged through the pipeline; downstream stages may read it and may not write back to it. Agent execution runs through a harness that enforces, in order: tenancy, schema-validated input, telemetry span open, execution, schema-validated output, span always closed. Model output that fails its output contract is rejected rather than persisted.

Progression, posture, and difficulty signals that shape the companion's behavior are **derived deterministically from durable records**, not inferred by a model — and only the posture-relevant projection of those signals crosses into the envelope. Raw scores stay out.

Retrieved user content is scanned on ingress for injection patterns before it reaches a prompt. Flagged chunks are score-penalized rather than deleted, because the archive is the user's and silently destroying their words to defend the model is the wrong trade.

### Honest degradation

When vector retrieval returns nothing above threshold, or times out, the system falls back to recent entries — and says so. The response is labeled as a fallback with reduced confidence, and the controls offered on fallback material differ from those offered on grounded material, so the UI never promises an action the backend cannot honor. Repeated retrieval failures trip a circuit breaker instead of hammering a degraded dependency.

### Durable asynchronous orchestration

AI work runs on queues consumed by a separate worker service: transcription, enrichment, reflection, archival, document ingestion, and classification each have their own lane. Job payloads are validated against per-queue key allowlists at enqueue time, so a malformed or over-broad payload never reaches a worker.

The interesting problem is not the queue — it is delivery truth. An in-process event emitter calling a listener is not an acknowledgement: once the process exits, nothing records whether the required downstream effect ever happened. The system distinguishes four states per required consumer — completed, never received, began then died, failed and still needs retry — by writing durable recovery intent **inside the same atomic document update** that marks the domain state as analyzed. There is no observable moment where the domain believes an event was emitted while no recovery intent exists. No payload is stored; replay is safe because the consumers are idempotent by unique index and upsert.

Stranded work is recovered by lease-and-reclaim: an owner takes a bounded lease on a retry, a crashed owner's lease expires, and periodic sweeps reclaim and re-drive the work. Retries converge on one outcome rather than racing to produce several.

### Observability without leaking the user

The AI pipeline is traced end to end — route handlers, GraphQL resolvers, workers, individual agent stages, and envelope assembly — with span metadata carrying retrieval strategy, candidate counts, how many passed threshold, confidence tier, and fallback state.

What it never carries is transcript text, retrieved excerpts, or user freeform content. Account identifiers reaching operator logs are reduced to a truncated, non-reversible digest: stable enough to correlate lines and count distinct owners, useless as an identity. Errors crossing into logs are sanitized to a fixed code enum rather than raw messages. In production, missing telemetry configuration fails the boot rather than silently disabling tracing.

### Lifecycle correctness for irreplaceable material

Audio has an explicit ownership lifecycle — pending, claimed, cleanup-owned, discarded — because a field trial proved that discarded takes were silently surviving as orphaned records and storage objects. Discard is ownership-checked and idempotent, an atomic pending-to-claimed guard makes a claim/discard race unable to orphan or delete a finalized record, and when storage cleanup fails the system reports the truthful non-deletion instead of fabricating success.

Each entry carries per-stage pipeline status plus a version stamp, so a record written before a stage existed is distinguishable from a record currently mid-flight through it. A missing transcript is not the same thing as a failed memory, and the system is built to know the difference.

---

## System Architecture

```text
Voice / text capture (browser)
        |
        v
Durable source artifact          <- object storage + entry record, written first
        |
        v
Async pipeline (queues + worker service)
   transcription -> enrichment -> reflection -> archival
        |
        v
Structured records + user-scoped vector memory
        |
        v
Governed retrieval
   threshold -> recency decay -> diversity re-rank -> veto/eligibility filters
        |
        v
Sealed context envelope
   injection scan -> deterministic derived signals -> frozen, read-only downstream
        |
        v
Bounded generation
   schema-validated in/out, claim type + confidence tier
        |
        v
Evidence-backed output + persisted receipts
        |
        v
Durable product state  (lifecycle, delivery acknowledgement, user veto controls)
```

Traced at every stage. Nothing on that path is allowed to block preservation of the source artifact.

---

## Selected Engineering Decisions

**Durable truth outranks ephemeral delivery.**
An event emitter invoking a listener proves nothing survived the process. Recovery intent is therefore written in the same atomic update that makes the domain state true, which removes the crash window a separate outbox write would have introduced — without needing a distributed transaction.

**Provenance is persisted, not reconstructed.**
Re-running retrieval when a user opens the receipts drawer would show them what the system *would* retrieve now, not what actually grounded the answer they are reading. Receipts are snapshotted onto the message at write time. This costs storage and buys the ability to be audited.

**Retrieval is governed, not raw.**
Nearest-neighbor over a personal archive is a repetition machine. Threshold, recency decay, diversity re-ranking, veto exclusion, and anti-repetition exclusion are all separate, individually testable policies — so retrieval quality can be tuned without touching generation.

**Async work must never block preservation.**
The source artifact is committed before any AI stage runs. Every downstream stage can fail, retry, or be skipped, and the user still has their memory. This inverts the usual pipeline instinct, where the artifact is the *output* of processing.

**The user can veto their own memory.**
An ownership-checked mutation permanently excludes a chunk from future retrieval. The system also refuses to advertise that control on material where it would be meaningless — offering an action the backend cannot honor is its own kind of lie.

**Telemetry excludes the interesting part.**
The most useful thing to log is the transcript. It is also the one thing that must never be logged. Traces carry retrieval shape and decision metadata instead, and identifiers are fingerprinted before they reach an operator log.

**AI decisions are subordinate to deterministic application state.**
Model output is schema-validated and rejected on contract violation. Progression, difficulty, and posture are derived from durable records rather than generated. The model composes language; it does not get to decide what is true.

---

## Tech Stack

| Layer | Tools |
|---|---|
| Frontend | React, Vite, Tailwind CSS, Framer Motion, Zustand |
| API | GraphQL (Apollo Server + Client), GraphQL subscriptions over WebSocket, Socket.IO |
| Backend | Node.js, Express |
| Data | MongoDB Atlas (document store + vector search), Mongoose |
| Object storage | AWS S3, CloudFront for public assets |
| Queue / async | BullMQ, Redis |
| AI | OpenAI (speech-to-text, embeddings, reflection), Anthropic Claude (enrichment) |
| Contracts | Zod schemas on agent and pipeline boundaries |
| Tooling protocol | Model Context Protocol SDK |
| Auth | Supabase (JWT verified server-side) |
| Observability | Langfuse |
| Deployment | Render — web service, background worker service, static site |
| Workflow | pnpm workspaces, GitHub Actions CI, Claude Code, Codex CLI |

Model selections are deliberately described by role rather than by SKU. Specific model versions are an operational detail that changes on a different clock than the architecture, and the architecture is the part worth showing.

---

## Public Showcase Boundary

This repository demonstrates architecture and engineering judgment. It is not a source release.

**Published here:** system shape, orchestration and lifecycle semantics, retrieval governance principles, provenance and evidence design, privacy posture, engineering decisions and their rationale.

**Kept private:** product source code, database schemas, prompt text, retrieval scoring formulas and calibrated constants, internal endpoints, credentials and environment configuration, user data, raw audio and transcripts, and trace payloads.

Any demonstration material is synthetic. Where a real implementation detail would function as a recipe rather than as evidence of judgment, it has been generalized on purpose.

---

## Engineering Practice

- **PR-based workflow** with a protected integration branch and explicit promotion PRs to `main`. Over two thousand merged pull requests to date, solo.
- **CI on every pull request:** environment-usage validation, GraphQL typedef parity, schema and operation validation, lint, tests, typecheck, build, and format checks. Separate workflows cover end-to-end browser tests, mutation testing, dependency audit, and performance budgets.
- **Evidence-based engineering.** Architectural changes are preceded by written audits that classify every claim as proven, needing proof, or explicitly not-to-be-claimed — including cataloguing the system's own ungoverned seams rather than quietly rounding them up to "governed."
- **Adversarial review.** AI-assisted implementation (Claude Code, Codex CLI) with automated review pinned to the exact current head, and bot findings verified before they are accepted. Several of the durability guarantees described above exist because a review caught a bypass path the original design had missed.
- **Regressions are treated as product problems, not chores.** The lifecycle work above began with a real-device field trial that disproved an assumption the code had been making for months.

---

## Current State

Active development with working end-to-end product paths.

Stable and verifiable today: voice capture through durable storage, asynchronous transcription and enrichment, user-scoped vector retrieval with governance filters, receipt-grounded companion responses with user-inspectable provenance, per-stage pipeline lifecycle with recovery of stranded work, and traced AI stages under a no-user-content telemetry contract.

This is a working system with real decisions at every layer, not a tutorial project — and it is still being built.

---

## About the Builder

**William L. Haynes** — founding engineer. Former ten-year English and ESOL teacher. Novelist.

A decade in the classroom trains a specific and underrated skill set: decomposing a complex system into legible steps, writing clearly under pressure, and telling the difference between what a person asks for and what they actually need. Those instincts turn out to be exactly what building and explaining a production AI system demands.

Echo Doj0 is what happens when a novelist's ear for language and an engineer's insistence on evidence work the same problem. The through-line of the work is a refusal to let a system claim more than it can prove — in the product, and in the documentation describing it.

Open to founding engineer, full-stack AI, and mission-driven product roles.

---

## Contact

- **Portfolio:** [williamhaynesportfolio.com](https://williamhaynesportfolio.com)
- **LinkedIn:** [linkedin.com/in/williamhaynesxp](https://www.linkedin.com/in/williamhaynesxp/)
- **GitHub:** [github.com/wileland](https://github.com/wileland)
- **Email:** wileland7@gmail.com
- **Phone:** (210) 775-8143
</content>
</invoke>
