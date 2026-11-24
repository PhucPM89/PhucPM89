<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="./assets/hero-dark.svg" />
    <source media="(prefers-color-scheme: light)" srcset="./assets/hero-light.svg" />
    <img alt="Phạm Minh Phúc — Backend & Distributed Systems" src="./assets/hero-dark.svg" width="100%" />
  </picture>
</p>

<p align="center">
  <img src="https://skillicons.dev/icons?i=cs,dotnet,go,postgres,redis,mongodb,kafka,rabbitmq,docker,githubactions&perline=10" alt="Core stack icons" />
</p>

---

### Focus

Code that works on the happy path is table stakes. I focus on the layer where distributed systems fail silently: **atomic state and event boundaries, concurrency under contention, fault isolation across unstable dependencies, and security-critical session lifecycles**.

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="./assets/architecture-dark.svg" />
    <source media="(prefers-color-scheme: light)" srcset="./assets/architecture-light.svg" />
    <img alt="Reference reliability pipeline: Write API, ACID State and Outbox, CDC Relay, Event Stream, Idempotent Inbox, and DLQ Replay" src="./assets/architecture-dark.svg" width="100%" />
  </picture>
</p>

---

### Systems Architecture & Engineering Depth

#### 1. Consistency & Messaging Guarantees (`Outbox/Inbox · CDC · Saga`)
* **Eliminating dual writes:** Database mutations and outgoing domain events must commit in the same local ACID transaction (`Transactional Outbox`). Publishing directly to Kafka or RabbitMQ inside or after a transaction creates an unrecoverable crash window.
* **Relay trade-offs (Poller vs. CDC):** Polling relays (`SELECT ... FOR UPDATE SKIP LOCKED` / indexed outbox tables) stay operationally simple at moderate throughput; WAL log-tailing (`Debezium CDC`) removes polling read load and preserves commit order at the cost of connector lag monitoring and strict schema-evolution discipline.
* **Effectively-once consumption:** Message brokers guarantee at-least-once delivery, not exactly-once domain effects. Consumers record processed `message_id` / idempotency keys inside an **Inbox table** within the same DB transaction as the state update.
* **Long-lived Sagas:** When workflows span services or external APIs where 2PC does not apply, I model transitions as explicit state machines—using Choreography for simple decoupled reactions and Orchestration when branching, SLA deadlines, and semantic compensations matter.

#### 2. Concurrency Control & Data Integrity (`Optimistic · Fencing · Isolation`)
* **Matching primitives to contention:** I avoid global locks. Use **Optimistic Concurrency** (`rowversion` / `xmin` + bounded jittered retries) when conflicts are rare; use **atomic conditional updates** (`WHERE balance >= @amount`) or **`SKIP LOCKED`** for hot queues and inventory reservations.
* **Why cache locks alone are unsafe:** A Redis lease can expire during a worker GC/CPU pause, allowing two workers to mutate the same aggregate concurrently. Distributed leases must be paired with monotonically increasing **fencing tokens** verified at the authoritative database write boundary.
* **Deliberate isolation levels:** Isolation (`Read Committed`, `Repeatable Read`, `Serializable`) is chosen per workflow invariant rather than left to ORM defaults.

#### 3. Resilience, Backpressure & Failure Isolation (`Timeouts · Circuit Breaker · DLQ`)
* **Slow brownouts kill faster than hard 500s:** Every outbound RPC (`gRPC` / HTTP) enforces strict per-hop timeouts, bounded retries with exponential backoff + full jitter, bulkheads, and Circuit Breakers to prevent thread-pool starvation from cascading.
* **Replayable failure handling:** Poison messages and exhausted retries route to **Dead Letter Queues (DLQ)** with correlation IDs and failure context so operators can inspect and safely replay without corrupting downstream state.
* **Measure before tuning:** Profile queries, allocation hotspots, and lock wait times before optimizing—never guess bottlenecks.

#### 4. Authentication, Session Security & Access Control (`MFA · Token Lifecycle · Claims`)
* **Cross-tab rotating refresh tokens:** Multi-tab apps trigger false token-reuse revocations when parallel tabs race to rotate the same refresh token. Coordinating refresh state across tabs with a client-side lock and a bounded server-side rotation grace window keeps security strict without breaking active sessions.
* **Step-up verification over blunt lockouts:** Combining layered `Role Claims + User Claims` with environment/fingerprint drift signals drives targeted **TOTP / SMS OTP** step-up challenges instead of silently trusting stale sessions or locking out legitimate users on minor browser changes.

#### 5. Bi-Directional Sync & Grounded AI Pipelines (`Webhooks · Reconciliation · RAG/MCP`)
* **Loop-free two-way integration:** Syncing external systems (e.g., issue trackers, ERPs) requires origin metadata + normalized payload hashes to kill webhook echo loops, paired with periodic cursor-based reconciliation sweeps and deterministic field-level ownership.
* **Production RAG & Tool Calling:** Treat LLM outputs as untrusted inputs until validated. Pair hybrid retrieval (`Qdrant` + metadata/tenant pre-filtering) with strict JSON Schema output enforcement, bounded tool-calling budgets (`MCP`), and provider abstraction for cost/quota control.

---

### Failure Mode Playbook

| Failure Scenario | Silent Risk | Architectural Defense |
| :--- | :--- | :--- |
| App crashes after DB commit, before broker publish | State exists locally; downstream services never react | **Transactional Outbox** in the same ACID commit + CDC/Poller relay |
| Broker redelivers message after slow consumer ack | Double deduction, duplicate side effects | **Idempotent Inbox** keyed by `message_id` + deterministic idempotency keys |
| Worker stalls past Redis lock TTL (GC / CPU pause) | Two workers write concurrently (split-brain) | **Monotonic fencing tokens** validated at DB write boundary |
| Parallel browser tabs refresh a rotating JWT together | False reuse detection revokes legitimate session | Cross-tab refresh coordination + brief rotation grace window |
| Downstream service stalls at 30s latency | Thread-pool exhaustion cascades across APIs | Strict per-hop timeouts + jittered retry budget + **Circuit Breaker** |
| Two-way webhook sync triggers self-updates | Infinite update loop & API rate-limit exhaustion | Origin marker + payload hash guard + scheduled reconciliation sweep |

---

### Engineering Trade-offs

* **Modular Monolith vs. Microservices:** Start with strict module boundaries inside one deployable unit while domain invariants evolve; extract services only for independent scaling, fault isolation, or team autonomy.
* **Synchronous gRPC/Protobuf vs. Async Kafka/RabbitMQ:** Use `gRPC` on critical paths requiring immediate invariant decisions; push cross-boundary side effects and slow workflows onto durable queues.
* **Debezium CDC vs. Polling Outbox:** Use polling for operational simplicity at moderate write volume; adopt CDC when polling IOPS or sub-second relay latency justifies operating WAL connectors.

---

### Technical Stack

| Layer | Technologies & Patterns |
| :--- | :--- |
| **Languages & Runtimes** | `C# / .NET` · `Go` · `Java` · `TypeScript` |
| **Data & Vector** | `PostgreSQL` · `SQL Server` · `Redis` · `MongoDB` · `Qdrant` |
| **Messaging & RPC** | `Apache Kafka` · `RabbitMQ` · `MassTransit` · `Debezium` · `gRPC` · `Protobuf` |
| **Reliability & Security** | `CQRS` · `Saga` · `Outbox/Inbox` · `Idempotency` · `DLQ` · `Circuit Breaker` · `MFA (TOTP/SMS)` |
| **AI & Delivery** | `LLM Integration` · `RAG` · `MCP` · `Tool Calling` · `Docker` · `GitLab CI/CD` · `GitHub Actions` |

---

<p align="center">
  <b>Contact:</b> <a href="mailto:minhphuc2308031@gmail.com">minhphuc2308031@gmail.com</a> ·
  <b>GitHub:</b> <a href="https://github.com/PhucPM89">github.com/PhucPM89</a>
</p>

<p align="center">
  <i>Engineering is about making failure modes explicit and complexity manageable.</i>
</p>
