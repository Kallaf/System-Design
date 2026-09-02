# System Design Interview Requirements & Clarifying Questions Bank

In system design interviews, the clarifying questions stage determines whether you build the right architecture. This cheat sheet serves as a categorized "question bank" and scoping framework to eliminate ambiguity within the first 5 minutes.

---

## 1. Quick Categorized Question Bank

| Category | High-Yield Clarifying Questions | Goal / Why Ask |
| :--- | :--- | :--- |
| **Functional Scope** | • What are the core 2–3 user features? <br>• Are we building end-to-end or focusing on specific components (e.g., ingestion vs. query)?<br>• What user roles/clients exist (e.g., Mobile, Web, Public API)? | Defines boundaries; prevents scope creep and wasted time. |
| **Traffic & Scale** | • What is the expected Daily Active Users (DAU) or Monthly Active Users (MAU)?<br>• What is the average and peak Read/Write requests per second (RPS)?<br>• What is the Read-to-Write ratio (e.g., 100:1 read-heavy vs. 1:1 write-heavy)? | Determines caching, sharding, and compute scaling strategies. |
| **Data & Volume** | • What is the size of a single payload/message?<br>• How long must data be retained (e.g., 30 days vs. forever)?<br>• What is the estimated total data growth over 1–5 years? | Dictates storage engines (NoSQL vs. Relational vs. Blob/S3). |
| **Performance & Latency** | • What is the SLA/SLO for read and write latencies (e.g., p99 < 100ms)?<br>• Is real-time processing required, or is eventual consistency / batch fine?<br>• Are there strict geographic latency requirements (multi-region setup)? | Determines sync vs. async flows and indexing strategies. |
| **Availability & Consistency** | • What is the target uptime availability (e.g., 99.9% vs. 99.999%)?<br>• Under the CAP theorem, do we prioritize Strong Consistency or High Availability?<br>• Can data be temporarily stale or dropped (e.g., analytics vs. payments)? | Directs replication strategy, consensus algorithms, and database choice. |
| **Security & Compliance** | • Does the system need end-to-end encryption or strict access control (RBAC)?<br>• Are there regulatory constraints (GDPR, PCI-DSS, HIPAA)? | Influences auth mechanisms, data masking, and multi-tenancy design. |

---

## 2. Deep-Dive Scoping Framework by System Type

### A. High-Throughput Read Systems (e.g., Newsfeed, Twitter, E-commerce Catalog)
* **Specific Questions:**
  * Is the feed personalized or global/chronological?
  * How do we handle celebrity/fan-out problems (users with millions of followers)?
  * How fast must a newly published post appear on a follower’s feed?
* **Key Scope Trade-off:** Push model (fan-out on write) vs. Pull model (fan-out on read) vs. Hybrid.

---

### B. High-Throughput Write Systems (e.g., Metric Logging, IoT, Analytics)
* **Specific Questions:**
  * Can we drop or sample data during peak spikes, or is zero-loss mandatory?
  * Is late-arriving or out-of-order data expected?
  * What are the primary aggregation query windows (e.g., 1-minute rollups vs. raw logs)?
* **Key Scope Trade-off:** In-memory queuing (Kafka) with LSM-tree storage engines vs. transactional relational databases.

---

### C. Real-Time & Streaming Systems (e.g., Chat, Gaming, Live Streaming, Collaborative Editing)
* **Specific Questions:**
  * Is communication 1-to-1, group-based, or broadcast?
  * What is the maximum group size (e.g., 50 people in a chat vs. 100,000 in a live channel)?
  * Should offline users receive push notifications and missed message sync?
* **Key Scope Trade-off:** Stateful WebSockets vs. stateless HTTP polling/SSE; message ordering guarantees.

---

### D. File / Media / Large Object Storage (e.g., YouTube, Google Drive, Image Upload)
* **Specific Questions:**
  * What is the maximum allowed file size?
  * Do we need video transcoding, image compression, or chunked chunk uploads?
  * Is content static or modified frequently after upload?
* **Key Scope Trade-off:** Direct client-to-Object Storage (S3) uploads via presigned URLs vs. routing through API gateways.

---

## 3. Estimations Cheat Sheet (Back-of-the-Envelope)

Use these standard rules of thumb when converting requirements into hard hardware constraints:

| Metric / Unit | Standard Scale Estimate | Useful Calculation Shortcuts |
| :--- | :--- | :--- |
| **Seconds per day** | $ pprox 86,400$ seconds | Round to **$10^5$ seconds/day** for quick mental math |
| **1 Million DAU** | Assuming $10$ actions/user/day | $= 10	ext{M requests/day}  pprox 115	ext{ RPS avg}$ (Peak = $2	ext{x–5x}  pprox 250	ext{–}500	ext{ RPS}$) |
| **100 Million DAU** | Assuming $10$ actions/user/day | $= 1	ext{B requests/day}  pprox 11,500	ext{ RPS avg}$ (Peak $ pprox 25,000	ext{–}50,000	ext{ RPS}$) |
| **Storage Math** | $1	ext{ KB} 	imes 1	ext{M requests}$ | $= 1	ext{ GB/day} = 365	ext{ GB/year}$ |
| **Storage Math** | $10	ext{ KB} 	imes 100	ext{M requests}$ | $= 1	ext{ TB/day} = 365	ext{ TB/year}$ |
| **Bandwidth Math** | $10,000	ext{ RPS} 	imes 10	ext{ KB/payload}$ | $= 100	ext{ MB/sec} = 800	ext{ Mbps}$ |

---

## 4. Step-by-Step Interview Execution Flow

```
+-----------------------------------------------------------------------+
| STEP 1: Understand Functional Requirements (~3 mins)                 |
| Ask: "What are the core 2-3 features?" & "Out-of-scope items?"        |
+-----------------------------------------------------------------------+
                                  |
                                  v
+-----------------------------------------------------------------------+
| STEP 2: Establish Non-Functional Requirements (~3 mins)               |
| Ask: DAU/RPS, Read/Write Ratio, Latency SLA, Consistency vs Availability|
+-----------------------------------------------------------------------+
                                  |
                                  v
+-----------------------------------------------------------------------+
| STEP 3: Back-of-the-Envelope Estimation (~2 mins)                    |
| Calculate: Memory/Cache size, Storage/year, Peak Write/Read RPS       |
+-----------------------------------------------------------------------+
                                  |
                                  v
+-----------------------------------------------------------------------+
| STEP 4: Define High-Level Architecture & APIs (~10 mins)             |
| Propose endpoints, database schema, and major component blocks         |
+-----------------------------------------------------------------------+
```

### Pro-Tip Scoping Checklist:
1. **Never guess—confirm:** If the interviewer gives a vague prompt like "Design Messenger," immediately ask: *"Are we focusing on 1-to-1 messaging, or do we also need group chats and online status indicators?"*
2. **State assumptions explicitly:** If the interviewer says "You decide," say: *"I will assume 10 million daily active users with a 10:1 read-to-write ratio, prioritizing low latency over strong consistency."*
