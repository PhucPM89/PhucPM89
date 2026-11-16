<h1 align="center">Phúc · Backend Engineer</h1>

<p align="center">
  <b>.NET · Distributed Systems · Reliability Engineering</b>
</p>

<p align="center">
  <i>I build backend systems that stay correct and predictable when things fail.</i>
</p>

<p align="center">
  <img src="https://skillicons.dev/icons?i=cs,dotnet,kafka,rabbitmq,postgres,redis,docker,githubactions&perline=8" />
</p>

---

## 🎯 Focus

I work on the part of backend engineering where "it works" is not enough:
data must stay consistent, messages must be neither lost nor processed twice,
and the system must degrade gracefully when a dependency goes down.

## 🧩 How I approach problems

**📨 Messaging without lost or duplicate events**
Dual writes to a database and a broker silently lose events. I use the
Transactional Outbox so state and events commit atomically, and idempotent
consumers so at-least-once delivery never produces duplicate side effects.

**🔒 Consistency and concurrency, chosen per workflow**
Optimistic concurrency where conflicts are rare and throughput matters.
Pessimistic locking only where contention is high and correctness is critical.
Isolation levels are a deliberate decision, not a default.

**🛡️ Failure is a design input**
Timeouts on every outbound call, retries with backoff and jitter, circuit
breakers around unstable dependencies, and idempotency keys on write endpoints
so clients can retry safely.

**⚡ Measure before optimizing**
Profile, find the real bottleneck, change one thing, measure again.

**🤖 LLM infrastructure**
Provider abstraction so vendors and models are swappable, with API key
management and per-key quota control to contain cost and abuse.

**📡 Real-time systems**
SignalR and WebSocket for live updates, Redis GEO for location queries.

## 🧰 Stack

| | |
|---|---|
| **Backend** | C# · .NET · ASP.NET Core · EF Core |
| **Messaging** | Kafka · RabbitMQ |
| **Data** | SQL Server · PostgreSQL · Redis |
| **Ops** | Docker · GitHub Actions · CI/CD |

---

## 📫 Contact

[![Email](https://img.shields.io/badge/Email-minhphuc2308031@gmail.com-D14836?style=flat&logo=gmail&logoColor=white)](mailto:minhphuc2308031@gmail.com)

<p align="center">
  <i>Engineering is about making complexity manageable.</i>
</p>
