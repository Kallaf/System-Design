# Deep Dive: Non-Functional Requirements (NFRs) & Architectural Techniques

In system design, functional requirements define **what** a system does, while non-functional requirements (NFRs) define **how well** the system performs. This deep dive covers key NFRs, how to quantify them, and the technical patterns and architecture strategies used to satisfy each requirement.

---

## Quick Reference Matrix

| Non-Functional Requirement | Common Metrics / Targets | Primary Architectural Techniques & Solutions |
| :--- | :--- | :--- |
| **High Availability & Reliability** | SLA %, Uptime (99.99%), MTBF, MTTR | Redundancy, Failover, Load Balancing, Circuit Breakers, Active-Active Clustering |
| **Scalability (Horizontal & Vertical)** | RPS / QPS, Throughput (MB/s), Concurrent Users | Sharding / Partitioning, Stateless Services, Read Replicas, Auto-scaling |
| **Low Latency & High Throughput** | p95/p99 Latency (<50ms), Throughput (TPS) | Multi-Level Caching, Asynchronous Processing, CDN, Indexing, Connection Pooling |
| **Consistency & Data Integrity** | RPO, Consistency Models (CAP / PACELC) | Consensus Protocols (Raft/Paxos), Distributed Transactions (Saga/2PC), Vector Clocks |
| **Fault Tolerance & Resilience** | Error Rate %, Graceful Degradation | Retries with Exponential Backoff + Jitter, Bulkheads, Dead Letter Queues (DLQ), Heartbeats |
| **Security & Compliance** | Audit Readiness, Encryption Overhead | Zero Trust Architecture, Encryption (At-rest & In-transit), Tokenization, RBAC/ABAC |
| **Maintainability & Observability** | MTTD, Deployment Frequency, Lead Time | Microservices, Distributed Tracing (OpenTelemetry), Metrics (Prometheus), CI/CD |
| **Cost Efficiency & Resource Utilization**| Compute Unit Cost, Infrastructure ROI | Serverless / Auto-scaling, Cold Storage Tiering, Resource Pooling, Compression |

---

## 1. High Availability & Reliability

### Core Metrics & Concepts
* **Availability:** Percentage of time the system remains operational and accessible under normal and adverse conditions (e.g., "four nines" = 99.99% uptime $ pprox$ 52.56 minutes of downtime per year).
* **Mean Time Between Failures (MTBF):** Average operating time between system failures.
* **Mean Time to Repair (MTTR):** Average time required to recover from a failure.

### Architectural Techniques & Solutions

#### Active-Active vs. Active-Passive Redundancy
* **Active-Passive (Primary-Standby):** Traffic is routed to a primary node; a secondary standby node takes over via VIP failover or DNS switching if the primary fails.
* **Active-Active:** Traffic is distributed across multiple operational nodes concurrently using a global load balancer (e.g., Anycast DNS or AWS Route 53).

#### Health Checks & Automated Failover
* **Heartbeating & Probes:** Load balancers continuously evaluate instance health via L4 (TCP connection) or L7 (`/healthz` HTTP endpoint) probes. Unhealthy nodes are removed from the active pool automatically.
* **Consensus-Based Failover:** Orchestration systems (e.g., Kubernetes, Consul) detect node crashes and automatically reschedule failed pods or elect new database primaries using algorithms like Raft.

#### Self-Healing Patterns
* **Circuit Breakers (e.g., Resilience4j):** Wraps external network calls. If error rates cross a defined threshold, the circuit "opens" immediately, failing fast to prevent cascading system collapse and giving downstream services time to recover.

---

## 2. Scalability

### Core Metrics & Concepts
* **Scalability:** The ability of a system to handle growing workloads by adding resources without degrading performance.
* **Horizontal Scaling (Scale-Out):** Adding more instances/nodes to the pool.
* **Vertical Scaling (Scale-Up):** Increasing RAM, CPU, or I/O capacity of existing machines.

### Architectural Techniques & Solutions

#### Database Sharding & Partitioning
* **Horizontal Partitioning (Sharding):** Distributes rows across multiple independent database instances using a **Shard Key** (e.g., `hash(user_id) % num_shards`).
* **Consistent Hashing:** Arranges nodes on a virtual ring using hash functions, minimizing data movement when scaling node count up or down (used in DynamoDB, Cassandra, Akamai CDN).

#### Stateless Microservices Design
* Offload session state and application data to central, high-speed stores (e.g., Redis, Memcached, database clusters).
* Allows any service instance to process any incoming request, enabling seamless auto-scaling (AWS HPA/KEDA) based on CPU/RAM/queue depth metrics.

#### Read-Write Splitting (Replication)
* Separate write queries to a **Primary DB** from read queries routed to multiple **Read Replicas**. This scales read traffic while isolating write locks.

---

## 3. Low Latency & High Throughput

### Core Metrics & Concepts
* **Latency:** Time taken to process a single request and return a response (measured in p50, p95, p99 percentiles).
* **Throughput:** Number of requests or data volume processed per unit of time (e.g., Queries Per Second ‚Äî QPS, Transactions Per Second ‚Äî TPS).

### Architectural Techniques & Solutions

#### Multi-Tier Caching Strategies
* **Client/CDN Edge Caching:** Cache static assets and immutable API payloads close to users (e.g., Cloudflare, CloudFront).
* **Application In-Memory Cache:** Use key-value stores (e.g., Redis, Memcached) to cache frequent read queries and computed aggregations.
* **Cache Read/Write Patterns:**
  * *Cache-Aside (Lazy Loading):* Application checks cache; if miss, reads from DB and populates cache.
  * *Write-Through:* Updates cache and DB simultaneously.
  * *Write-Behind (Write-Back):* Writes to cache first, asynchronously flushing updates to DB in batches.

#### Asynchronous Processing & Task Queues
* Convert heavy blocking workflows (e.g., generating PDFs, sending emails, video processing) into asynchronous tasks using message brokers (e.g., RabbitMQ, AWS SQS, Celery).

#### Database Query Optimization
* **Indexes:** B-Trees for range queries, Hash indexes for point lookups, GIN/GiST for full-text/geospatial search.
* **Denormalization:** Read-optimize relational schemas to eliminate expensive multi-table joins.
* **Connection Pooling:** Reuse pre-established database TCP connections (e.g., PgBouncer) to avoid connection setup overhead.

---

## 4. Consistency & Data Integrity

### Core Metrics & Concepts
* **Consistency:** Ensuring all nodes see the same data at the same time, or defining acceptable trade-offs (Strong Consistency vs. Eventual Consistency).
* **Recovery Point Objective (RPO):** Maximum acceptable age of files/data recovered from storage after a failure.
* **CAP & PACELC Theorems:** Trade-offs between Consistency, Availability, Partition Tolerance, Latency, and Else conditions.

### Architectural Techniques & Solutions

#### Distributed Consensus Protocols
* Use protocols like **Raft**, **Paxos**, or **Zab** to maintain consistent distributed state (e.g., etcd, ZooKeeper) across nodes even during network partitions.

#### Distributed Transactions & Saga Pattern
* **Two-Phase Commit (2PC):** Strongly-consistent blocking protocol across distributed databases; high latency, vulnerable to coordinator failure.
* **Saga Pattern (Choreography / Orchestration):** Event-driven chain of local transactions. If a step fails, compensating transactions execute backwards to restore data consistency.

#### Vector Clocks & Conflict Resolution
* Track causality in eventual-consistency systems (e.g., Cassandra, Riak). Use techniques like **Last-Write-Wins (LWW)** or CRDTs (Conflict-Free Replicated Data Types) to resolve write conflicts automatically.

---

## 5. Fault Tolerance & Resilience

### Core Metrics & Concepts
* **Fault Tolerance:** The ability of a system to continue operating without interruption during component failures.
* **Blast Radius Minimization:** Designing boundaries to prevent localized failures from spreading.

### Architectural Techniques & Solutions

#### Retry Mechanisms with Exponential Backoff & Jitter
* Automatically retry transient network failures using progressively longer delay intervals ($2^n$ seconds) modified with random **jitter** to prevent thundering herd problems on recovering downstream services.

#### Bulkhead Isolation
* Segregate system resources (e.g., separate thread pools, memory limits, process groups) so that a failure in one subsystem cannot starve resources for the entire platform.

#### Dead Letter Queues (DLQ) & Poison Pill Handling
* Route malformed or repeatedly failing messages to a separate queue for offline manual inspection, keeping the primary message processing pipeline clear.

---

## 6. Security & Compliance

### Core Metrics & Concepts
* **Confidentiality, Integrity, Availability (CIA Triad):** Core security tenets.
* **Compliance Standards:** GDPR, HIPAA, PCI-DSS, SOC 2.

### Architectural Techniques & Solutions

#### Zero Trust & Network Segmentation
* Enforce authentication and authorization for every request, regardless of whether it originates inside or outside the network boundary. Use **Mutual TLS (mTLS)** via service meshes (e.g., Istio, Linkerd) for service-to-service communication.

#### Data Encryption
* **In-Transit:** TLS 1.3 for network transport.
* **At-Rest:** AES-256 for persistent database storage, backups, and block volumes.
* **Field-Level Encryption / Tokenization:** Encrypt sensitive fields (e.g., credit cards, PII) before writing to database storage.

#### Fine-Grained Access Control
* Combine **Role-Based Access Control (RBAC)** and **Attribute-Based Access Control (ABAC)** using policy engines (e.g., Open Policy Agent - OPA).

---

## 7. Maintainability & Observability

### Core Metrics & Concepts
* **Observability:** Ability to infer internal system states based on external outputs (Telemetry).
* **Mean Time to Detect (MTTD):** Time taken to identify an incident.

### Architectural Techniques & Solutions

#### The Three Pillars of Observability
* **Metrics (Prometheus / Grafana):** Aggregated numerical data points tracking system health over time (CPU, RAM, latency, throughput).
* **Logs (ELK / OpenSearch / Loki):** Structured context-rich logs tagged with correlation IDs.
* **Distributed Tracing (OpenTelemetry / Jaeger):** Pass unique trace/span headers across microservices to visualize request paths and identify performance bottlenecks.

#### Deployment Strategies
* **Blue-Green Deployments:** Maintain two identical environments; cut over traffic atomically via load balancer once testing passes.
* **Canary Releases:** Gradually shift a small percentage of real user traffic (e.g., 5%) to a new version to monitor error rates before full rollout.

---

## 8. Cost Efficiency & Resource Optimization

### Core Metrics & Concepts
* **Cost Efficiency:** Maximizing system throughput and capability per dollar spent on cloud resources.

### Architectural Techniques & Solutions

#### Tiered Storage Strategies
* Segregate data into Hot (SSD/In-Memory), Warm (Standard HDD/S3 Standard), and Cold/Archive (S3 Glacier) storage tiers based on access frequency.

#### Resource Pooling & Serverless Auto-Scaling
* Utilize serverless computing (e.g., AWS Lambda, Cloud Run) or container auto-scalers (KEDA) to scale resources to zero when idle, minimizing base operating costs.
