# Fundamentals of System Design

This document covers the **core foundations** of system design.

The goal is not memorization, but **clear thinking under constraints**.

---

## 1. What Is System Design?

System design is the process of breaking a problem into components and deciding **how they interact** to meet functional and non-functional requirements under real-world constraints.

System design answers questions like:

- How does data flow through the system?
- Where are the bottlenecks?
- What fails first at scale?
- How does the system evolve as traffic grows?

**Key idea:**

> System design is decision-making under uncertainty. There is no perfect design — only trade-offs.

---

## 2. Functional vs Non-Functional Requirements

### Functional Requirements (What the system does)

These define the core features of the system.

Examples:

- Users can shorten a URL
- Users can send messages
- Payments can be processed

### Non-Functional Requirements (How well it does it)

These define constraints and quality attributes.

Examples:

- Handle 1M requests per day
- Response time under 200ms
- 99.99% availability
- Secure and compliant

**Why this matters:**
Most poor designs fail because architecture decisions are made **before** non-functional requirements are clear.

---

## 3. Latency vs Throughput

### Latency

Latency is the time taken to process a **single request**.

Example:

- API response time = 120ms

### Throughput

Throughput is the number of requests the system can handle per unit time.

Example:

- 10,000 requests per second

**Trade-off:**
Improving throughput (e.g. batching) often increases latency, and reducing latency may reduce throughput.

**Analogy:**

- Latency = time taken by one customer at a billing counter
- Throughput = customers served per hour

---

## 4. Availability vs Reliability

### Availability

Availability measures whether the system is **up and reachable**.

Typical targets:

- 99.9% → ~8.7 hours downtime/year
- 99.99% → ~52 minutes downtime/year

### Reliability

Reliability measures whether the system **behaves correctly over time**.

A system can be available but unreliable (e.g. up but dropping messages).

**Key idea:**
Availability is about uptime. Reliability is about correctness.

---

## 5. Scalability vs Performance

### Performance

Performance describes how fast the system responds under current load.

Example:

- API responds in 50ms

### Scalability

Scalability describes the system’s ability to **maintain performance as load increases**.

A system can perform well initially but fail to scale due to architectural limits.

---

## 6. Vertical vs Horizontal Scaling

### Vertical Scaling (Scale Up)

Increasing the capacity of a single machine.

Examples:

- More CPU
- More RAM

Pros:

- Simple

Cons:

- Hardware limits
- Single point of failure

---

### Horizontal Scaling (Scale Out)

Adding more machines to distribute load.

Examples:

- Multiple API servers behind a load balancer

Pros:

- High scalability
- Fault tolerance

Cons:

- Increased system complexity

**Rule of thumb:** Start with vertical scaling, but design for horizontal scaling.

---

## 7. Stateless vs Stateful Systems

### Stateless Systems

The server does not store client state between requests.

Examples:

- REST APIs
- JWT-based authentication

Pros:

- Easy to scale
- Load balancer friendly

Cons:

- Requires external state storage

---

### Stateful Systems

The server stores client state locally.

Examples:

- In-memory sessions
- WebSocket connections

Pros:

- Faster access to state

Cons:

- Hard to scale
- Requires sticky sessions

**Key question:** Where does the state live?

---

## 8. CAP Theorem (Practical View)

In a distributed system, you can guarantee only two of the following three:

- Consistency: All nodes see the same data
- Availability: The system always responds
- Partition Tolerance: The system survives network splits

Since network partitions are unavoidable, systems must choose between:

- CP (Consistency + Partition Tolerance)
- AP (Availability + Partition Tolerance)

Examples:

- Banking systems → CP
- Social media feeds → AP

CAP is not a design choice — it is a constraint imposed by reality.

---

## 9. Idempotency

Idempotency means that repeating the same request produces the **same result**.

Example:

- Retrying a payment request should not charge the user twice

**Why it matters:**

- Network retries
- Client timeouts
- Message queue reprocessing

Common techniques:

- Idempotency keys
- Request IDs
- Deduplication tables

---

## Final Notes

If you deeply understand these fundamentals, you can:

- Reason clearly about system trade-offs
- Avoid common scaling mistakes
- Design systems that evolve gracefully

Everything else in system design builds on these principles.
