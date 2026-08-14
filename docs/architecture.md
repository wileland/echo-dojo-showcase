# Echo Doj0 — Architecture

A technical companion to the [README](../README.md). Where the README explains *what* was built and why it was hard, this document explains *how the system is structured* and which invariants hold it together.

Public-safe by construction: it describes decisions and contracts, not implementations. No source, schemas, prompts, scoring formulas, calibrated constants, endpoints, credentials, or user data appear here.

---

## 1. The Question This Architecture Answers

Echo Doj0 records a person's voice, preserves it, understands it asynchronously, and later reflects it back with evidence.

Almost every hard decision in the system descends from one asymmetry:

> **The user's source material is irreplaceable. Everything the AI derives from it is reproducible.**

A transcript can be regenerated. An embedding can be recomputed. A reflection can be re-run. The forty seconds someone spoke at 2am cannot. So the architecture treats the source artifact as the load-bearing thing and every derived artifact as disposable, retryable, and subordinate.

The rest of this document is that asymmetry applied to storage, orchestration, retrieval, generation, failure, and telemetry.

---

## 2. Layers and Ownership

| Layer | Owns |
|---|---|
| Client | Capture, review, retrieval surfaces, provenance inspection, lifecycle-aware UI states |
| API | Authentication boundary, data access contracts, ingestion, mutation authorization |
| Durable store | Application truth: entries, lifecycle state, derived signals, persisted receipts |
| Object storage | Durable audio artifacts |
| Vector memory | User-scoped retrievable chunks with integrity and provenance fields |
| Queues + worker service | Asynchronous derivation: transcription, enrichment, reflection, archival, ingestion |
| Orchestration | Stage sequencing, durable delivery acknowledgement, lease-based recovery |
| Retrieval governance | Threshold, decay, diversity, eligibility, veto, anti-repetition |
| Context assembly | Sealed envelope construction, ingress injection scanning, deterministic signal derivation |
| Observability | Stage-level tracing under a strict no-user-content contract |

The API process and the worker process are deployed separately. This is not incidental: it makes "the web tier is healthy but derivation is backed up" a state the system can be in, observe, and report, rather than a state it silently becomes.

---

## 3. What Is Durable

Durability questions are answered in one place, not negotiated per feature.

**Durable application truth lives in the database and object storage.** The entry record, its per-stage pipeline status, its audio lifecycle state, its derived enrichment signals, and the receipts persisted onto generated messages are the system of record.

**Transport is not truth.** Sockets, in-process event emitters, and queue acknowledgements move work; they do not record that work happened. Every guarantee the product makes is anchored in a durable write, and every ephemeral channel is treated as an optimization over a state the client could have fetched anyway.

**Ordering matters more than speed on the write path.** The durable source artifact and its entry record are committed before any derivation is scheduled. There is no configuration in which a failure in transcription, enrichment, reflection, or indexing can cost the user their recording.

---

## 4. What Is Asynchronous

Everything the AI does.

Capture returns as soon as the source is durably preserved. Derivation runs on queues consumed by the worker service, with separate lanes for transcription, enrichment, reflection, archival, document ingestion, and classification. Job payloads are validated against per-queue key allowlists at enqueue time, so a lane cannot be handed a shape it was never designed to accept and a malformed payload never reaches worker code.

Each entry carries **per-stage lifecycle status** rather than a single boolean. The client distinguishes idle, in-flight, completed, skipped, and failed per stage, and a version stamp on the record separates "written before this stage existed" from "currently mid-flight through it." A missing transcript is not a failed memory; a skipped stage is not an error; a stalled stage is not a completed one. The UI is built to say which.

---

## 5. How AI Work Enters the System

AI never enters through a free-floating call. It enters through three narrow, contract-enforced doors.

**The worker door.** A queued job for a specific stage, with a validated payload, executing against a durable record it re-reads rather than trusts from the message.

**The harness door.** Agent functions execute inside a runner that enforces a strict order: tenancy check, schema-validated input, telemetry span opened, execution, schema-validated output, span closed in a `finally` so spans can never hang open. Tenancy is the harness's responsibility and deliberately does not appear inside any input schema — there is exactly one source of truth for whose data is being touched. Output that violates its contract is rejected, not persisted.

**The tool door.** Model-facing tools are registered behind a registry with trust assertions on the calls that reach it. The retrieval perimeter is periodically audited, with every read and write path classified by governance tier and known limitations tracked explicitly rather than silently treated as governed. That discipline is the point: an audit whose output is always "everything is fine" is not an audit, and a limitation that has been written down and prioritized is a different kind of risk than one nobody has looked for.

---

## 6. How Memory Is Retrieved

Retrieval is a policy pipeline. Each stage is separately testable, and generation quality can be tuned by changing retrieval policy without touching generation code.

```text
user-scoped vector query
    -> similarity threshold          (below-floor candidates discarded)
    -> recency decay                 (gentle tiebreaker, not a dominant suppressor)
    -> diversity-aware re-ranking    (avoid five near-duplicates of one memory)
    -> eligibility + veto filters    (user vetoes, retrieval-only material)
    -> anti-repetition exclusion     (bounded; recently surfaced chunks)
    -> integrity verification        (hash check before anything becomes a receipt)
```

Several properties are worth naming explicitly.

**The canonical vector-retrieval path requires user scope before execution and fails closed when scope is absent.** Scope is applied inside the vector-query boundary rather than as a post-filter in application code, so the narrowing is part of the search rather than a step a caller has to remember. Enforcement is centralized at that boundary rather than reimplemented per call site, which is what makes it auditable.

**The exclusion set is bounded.** Anti-repetition state grows without limit as a conversation continues. Passing all of it into the query would eventually exceed the search engine's clause limits — a failure that would present as a broken product rather than a degraded one. Only the most recent bounded window is passed down.

**Over-fetching is capped.** Because governance filters remove candidates after the query returns, the system may need to fetch more than it keeps. That loop has explicit attempt and fetch ceilings, and it terminates when a pass discovers no new candidates. Filtering never degrades into an unbounded scan.

**Vector queries have a deadline.** A timeout produces an explicit degraded state, not a hang and not a silent empty result.

**Reference material is structurally ineligible to be a receipt.** Documents ingested to inform retrieval are marked at ingestion time and cannot be surfaced to the user as if they were the user's own words. This is enforced by fail-closed equality, so an absent marker excludes rather than admits.

---

## 7. How Evidence Is Attached to Claims

This is the part the product is actually about.

**Verification precedes citation.** Retrieved text is re-hashed and compared against the hash recorded when that memory was written. A mismatch drops the candidate and logs a warning containing no user content. Only integrity-verified material becomes a receipt. The provenance guarantee is therefore a property of the data, not a property of a prompt instruction.

**Claims are typed.** Every response carries a claim type — grounded in verified retrieved memory, falling back to recency, or empty — plus a confidence value and a surfacing rule explaining *why* this material was shown. A confidence tier is derived from those, and generation is conditioned on it. The system's language changes when its evidence changes.

**Receipts are persisted at write time.** Claim type, confidence, surfacing rule, and the cited excerpts are written onto the assistant message as it is created. When the user later opens the provenance drawer, they are reading a durable snapshot of what actually grounded the response — not the result of re-running a search that may now return something different. Reconstructed provenance is not provenance.

**Controls match the evidence.** A user can permanently veto a memory chunk through an ownership-checked mutation, after which it is excluded at the query filter for all future retrieval. That control is deliberately *not* offered on fallback material, where it would be meaningless. Advertising an action the backend cannot honor is a trust failure, however small.

---

## 8. How Context Reaches the Model

Context is assembled exactly once, into a **sealed envelope**, and passed unchanged through the remaining stages. The envelope is frozen: downstream stages may read every field and may write none. Assemble-once-pass-unchanged removes an entire category of bug in which two stages disagree about what the model was told.

Two things happen during assembly.

**Ingress scanning.** Retrieved user content is untrusted input on its way into a prompt — an archive can contain text that reads as an instruction, whether adversarially or accidentally. Chunks matching injection patterns are **score-penalized rather than dropped**. Deleting a user's own words to protect the model would be the wrong trade in a product whose entire premise is that the archive is theirs; reducing that material's influence on ranking is the right one.

**Deterministic signal derivation.** Progression, posture, and difficulty signals that shape the companion's behavior are computed from durable records — not generated by a model. Each is derived independently, each degrades gracefully to absent on failure rather than throwing, and each contributes only a narrow posture-relevant projection to the envelope. Raw scores and internal vectors never cross the boundary. Where the product has committed to a signal never being shown as a number, the number does not enter the envelope at all, so no prompt can accidentally surface it.

The generation stage then works from typed evidence and validated inputs, and its output is schema-checked before it becomes durable. **The model composes language; deterministic application state decides what is true.**

---

## 9. What Happens When Downstream Work Fails

The system is designed around the assumption that stages fail and processes die.

**The source survives.** Always. Every failure mode below is a failure to derive, never a failure to preserve.

**Delivery is acknowledged durably, per consumer.** An in-process emitter invoking a listener is not an acknowledgement — once the process exits, nothing records whether the required downstream effect happened. The system distinguishes four states per required consumer: completed, never received, began but the process died, and failed and still requires retry.

It does this by writing durable recovery intent **inside the same atomic document update** that makes the domain state true. There is no observable moment in which the domain considers the event emitted while no recovery intent exists — which is the linearization point the whole design turns on, and the reason it needs neither a distributed transaction nor a separate outbox collection with its own crash window. No payload is stored; only the intent. Replay is safe because the required consumers were already idempotent by unique index and upsert, and that pre-existing duplicate safety is precisely what makes replay legal.

**Stranded work is reclaimed by lease.** An actor takes a bounded lease before attempting a durable retry. A crashed actor's lease expires and periodic sweeps reclaim and re-drive the work. Leases are long enough that an ordinary in-flight call is never reclaimed out from under itself, and short enough that genuinely abandoned work does not sit stranded. Ownership is checked on release, so settling a superseded attempt can never clobber a newer owner.

**Terminal failures are terminal.** Errors that retrying cannot fix are marked unrecoverable rather than retried forever, and they are reconciled: a stage that will never complete transitions the record into a coherent state instead of leaving it ambiguous.

**Errors are sanitized before they travel.** Failures crossing into logs, telemetry, or client-visible status are reduced to a fixed code enum. Raw error text — which can carry database content or user material — does not propagate.

**Race safety is explicit where it matters.** Audio moves through a pending → claimed → cleanup-owned → discarded lifecycle with an atomic pending-to-claimed guard, so a concurrent claim and discard cannot orphan a record or delete the artifact backing a finalized one. When storage cleanup fails, the system reports the truthful non-deletion rather than fabricating success — a discarded-looking record whose bytes still exist is worse than an honest error.

---

## 10. What State Owns Truth

| Question | Authoritative source |
|---|---|
| Does this memory exist? | The durable entry record and its object-storage artifact |
| How far has processing gotten? | Per-stage pipeline status on the record, plus its version stamp |
| Did a required downstream effect happen? | Durable per-consumer delivery acknowledgement on the record |
| What grounded this response? | Receipts persisted onto the message at write time |
| Is this memory retrievable? | Chunk eligibility and veto state, enforced at the query filter |
| What is the user's progression or posture? | Deterministic derivation from durable records |
| What did the model say? | The persisted message — never re-generated to answer a later question |

Nothing on this list is owned by a socket, a cache, an in-flight job, or a model.

---

## 11. How Users Are Isolated

User scope is a security boundary, treated with the seriousness that word implies.

- Authentication is verified server-side at the API boundary; identity is never taken from client-supplied fields.
- Retrieval scope is applied inside the vector query filter, and the single enforcement point refuses to run without it.
- The agent harness enforces tenancy before any schema parsing or execution, and tenancy deliberately lives outside the input schema so it cannot be spoofed by a payload.
- Mutations affecting a user's own memory — veto, discard, claim — verify ownership of the underlying record before acting.
- Storage cleanup selects its backend and target from trusted persisted fields only, never from mutation input, so a client cannot direct a deletion.

---

## 12. How Observability Works Without Leaking

The AI pipeline is traced end to end: route handlers, GraphQL resolvers, worker jobs, individual agent stages, and envelope assembly each emit spans.

Span metadata carries the shape of a decision, never its content: retrieval strategy, candidate counts, how many passed threshold, confidence and confidence tier, fallback state, stage timing, and outcome. It does not carry transcript text, retrieved excerpts, generated prose, or user freeform input.

Supporting rules make that contract hold under pressure:

- Payloads are passed through a sanitization layer before leaving the process.
- Account identifiers reaching operator logs are reduced to a truncated, non-reversible digest — stable enough to correlate lines and count distinct owners, useless as an identity.
- Errors are sanitized to codes before they are traced.
- Spans are closed in `finally`, so a failure produces a complete trace rather than a dangling one.
- In production, missing telemetry configuration fails the boot. Observability is a deployment requirement, not a best-effort side channel that can silently be off when it is most needed.

The design pressure here is worth stating plainly: in a system like this, the single most useful thing to log is exactly the thing that must never be logged. The architecture resolves that by making decision metadata rich enough that nobody needs the payload to debug.

---

## 13. Where Agents Are Bounded

"Agentic" is a claim that needs qualifying, so here is the precise scope.

**What is true:** tool-using, schema-bounded AI stages run inside a harness with enforced tenancy, validated input and output contracts, telemetry, and deterministic sequencing. Tools are registered behind a registry with trust assertions. Retrieval is governed by policy the model does not control. Context is sealed before generation. Derived state that shapes behavior is computed, not generated. Model output that fails its contract is rejected rather than persisted.

**What is not claimed:** there is no autonomous multi-agent system, no self-directed planner with open-ended authority, and no path by which a model decides what becomes durable application truth. Agents run bounded work inside a pipeline that deterministic application code sequences and validates.

The boundary is the design. A system holding this kind of material should not have an autonomous agent with write authority over it, and this one does not.

---

## 14. Public / Private Boundary

**This repository publishes:** system shape, layer ownership, durability and lifecycle semantics, retrieval governance principles, provenance design, failure and recovery model, privacy posture, and the reasoning behind each.

**The private repository retains:** source code, database schemas, prompt text, scoring formulas and calibrated retrieval constants, index configuration, internal endpoints and hostnames, credentials and environment configuration, user data, raw audio and transcripts, and trace payloads.

Where an implementation detail would function as a recipe rather than as evidence of judgment, it has been generalized deliberately. The goal of this document is to let a senior engineer evaluate the decisions — not to let anyone reproduce the product.

---

## 15. What This Architecture Demonstrates

| Area | Signal |
|---|---|
| Production AI systems | Governed retrieval, sealed context assembly, bounded agent execution, evidence-typed generation |
| Distributed correctness | Durable delivery acknowledgement, atomic linearization points, lease-based recovery, idempotent replay |
| Failure engineering | Honest degradation, circuit breaking, terminal-failure reconciliation, race-safe lifecycle transitions |
| Trust and provenance | Hash-verified receipts, write-time provenance snapshots, user veto as a first-class control |
| Privacy under pressure | No-user-content telemetry contract, identifier fingerprinting, error sanitization, boot-time enforcement |
| Full-stack engineering | React client, Node/Express API, GraphQL contracts, document + vector data modeling, media ingestion, realtime |
| Product judgment | Source material outranks derived output; the system says what it can prove and labels what it cannot |
| Communication | Decisions explained with their tradeoffs, and claims scoped to what the implementation actually supports |
