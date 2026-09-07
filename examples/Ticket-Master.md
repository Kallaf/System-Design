# Ticket Master

## Requirements & Clarifying

| Category | High-Yield Clarifying Questions | Answers |
| :--- | :--- | :--- |
| **Functional Scope** | • What are the core 2–3 user features? <br>• Are we building end-to-end or focusing on specific components (e.g., ingestion vs. query)?<br>• What user roles/clients exist (e.g., Mobile, Web, Public API)? | Defines boundaries; prevents scope creep and wasted time. |
| **Traffic & Scale** | • What is the expected Daily Active Users (DAU) or Monthly Active Users (MAU)?<br>• What is the average and peak Read/Write requests per second (RPS)?<br>• What is the Read-to-Write ratio (e.g., 100:1 read-heavy vs. 1:1 write-heavy)? | Determines caching, sharding, and compute scaling strategies. |
| **Data & Volume** | • What is the size of a single payload/message?<br>• How long must data be retained (e.g., 30 days vs. forever)?<br>• What is the estimated total data growth over 1–5 years? | Dictates storage engines (NoSQL vs. Relational vs. Blob/S3). |
| **Performance & Latency** | • What is the SLA/SLO for read and write latencies (e.g., p99 < 100ms)?<br>• Is real-time processing required, or is eventual consistency / batch fine?<br>• Are there strict geographic latency requirements (multi-region setup)? | Determines sync vs. async flows and indexing strategies. |
| **Availability & Consistency** | • What is the target uptime availability (e.g., 99.9% vs. 99.999%)?<br>• Under the CAP theorem, do we prioritize Strong Consistency or High Availability?<br>• Can data be temporarily stale or dropped (e.g., analytics vs. payments)? | Directs replication strategy, consensus algorithms, and database choice. |
| **Security & Compliance** | • Does the system need end-to-end encryption or strict access control (RBAC)?<br>• Are there regulatory constraints (GDPR, PCI-DSS, HIPAA)? | Influences auth mechanisms, data masking, and multi-tenancy design. |

---
