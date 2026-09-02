# Comprehensive Database Selection Guide for System Design Interviews

Selecting the correct database architecture is one of the most critical decisions in system design interviews. This guide outlines the major database paradigms, their inner workings, trade-offs, and clear decision frameworks.

---

## 1. Quick Decision Matrix

| Database Type | Primary Use Case | Key Characteristics | Examples |
| :--- | :--- | :--- | :--- |
| **Relational (RDBMS)** | ACID transactions, structured data, complex joins | Strong consistency, schema-enforced, SQL support | PostgreSQL, MySQL, CockroachDB |
| **Key-Value** | Caching, session storage, real-time lookups | High throughput, sub-millisecond latency, simple KV operations | Redis, AWS DynamoDB, Memcached |
| **Document Store** | Semi-structured data, rapid prototyping, catalog management | Schema-flexible, nested JSON objects, rich query capabilities | MongoDB, Couchbase |
| **Wide-Column (Columnar)** | Heavy write volume, time-series, massive scale analytics | Column-family oriented, high write throughput, tunable consistency | Apache Cassandra, ScyllaDB, HBase |
| **Graph Database** | Highly interconnected data, social networks, fraud detection | Index-free adjacency, native relationship traversal | Neo4j, AWS Neptune, Amazon Neptune |
| **Search Engine / Inverted Index** | Full-text search, log analytics, fuzzy filtering | Inverted index structure, fast aggregation, relevance scoring | Elasticsearch, OpenSearch, Meilisearch |
| **Time-Series Database (TSDB)** | Metrics, IoT telemetry, financial tick data | Time-bucketed storage, fast append, automated data retention | InfluxDB, TimescaleDB, Prometheus |
| **Vector Database** | AI/ML embeddings, semantic search, recommendation systems | Nearest neighbor search (HNSW, IVF), high-dimensional indexing | Pinecone, Milvus, Qdrant |

---

## 2. Detailed Database Paradigms & Trade-Offs

### Relational Databases (RDBMS)
Relational databases organize data into tables with predefined schemas. They enforce ACID (Atomicity, Consistency, Isolation, Durability) guarantees, making them ideal for systems where data integrity is non-negotiable.

* **When to Use:**
  * Financial systems, payment processing, and order management.
  * Systems requiring complex multi-row joins and transactions.
  * Structured data with predictable schema requirements.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Strong ACID guarantees ensure strict consistency. | Vertical scaling limits (harder to scale horizontally). |
| Standardized query language (SQL). | Rigid schemas require expensive migrations at scale. |
| Rich ecosystem, mature tooling, and indexing options. | Poor performance with highly nested or unstructured data. |

---

### Key-Value Stores
Key-Value databases store data as a collection of key-value pairs. They operate like a massive distributed hash table, offering fast lookup operations ($O(1)$ complexity).

* **When to Use:**
  * Caching layers (e.g., caching database queries or API responses).
  * Session management, user shopping carts, and rate limiting.
  * Leaderboards and temporary token storage.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Extremely low latency (sub-millisecond reads/writes). | Limited querying capabilities (lookup by key only). |
| High throughput and trivial horizontal scaling. | Poor fit for complex relationships or multi-field queries. |
| Simple data model and lightweight overhead. | Memory constraints if used purely in-memory (e.g., Redis). |

---

### Document Stores
Document databases store semi-structured data formatted as JSON, BSON, or XML. Data that belongs together is typically stored together, reducing the need for joins.

* **When to Use:**
  * Content management systems (CMS), user profiles, and product catalogs.
  * Mobile applications with rapidly evolving features and dynamic schemas.
  * Polymorphic data models where records don't share identical fields.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Flexible, dynamic schema allows fast iteration. | Risk of data redundancy and duplicate storage. |
| Intuitive mapping to object-oriented programming code. | Weaker transaction support across multiple documents/collections. |
| Built-in horizontal sharding and auto-partitioning. | Potential for inconsistent data if not managed at the application layer. |

---

### Wide-Column (Columnar) Stores
Wide-column stores organize data into column families rather than rows. They are optimized for high-write throughput and massive dataset distribution across clusters.

* **When to Use:**
  * Write-heavy applications like activity tracking, messaging history, and IoT logs.
  * Multi-datacenter, high-availability write pipelines.
  * Large-scale data platforms requiring linear horizontal scalability.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Exceptional write performance (LSM-tree based storage engines). | Query flexibility is limited; queries must align with primary keys. |
| Seamless multi-region replication and fault tolerance. | No native support for multi-table joins. |
| Scales horizontally to petabytes of data seamlessly. | Steeper learning curve for data modeling and query design. |

---

### Graph Databases
Graph databases represent data as nodes (entities), edges (relationships), and properties. They use **index-free adjacency**, meaning each node maintains direct pointers to adjacent nodes.

* **When to Use:**
  * Social networks (friend graphs, follower networks).
  * Fraud detection networks and identity resolution.
  * Knowledge graphs, recommendation engines, and supply chain tracking.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Traverses complex, multi-hop relationships in constant time. | Poor performance for global aggregation operations. |
| Highly expressive query languages (e.g., Cypher, Gremlin). | Difficult to shard horizontally across large distributed clusters. |
| Flexible structure for adding new relationship types. | Higher resource cost compared to traditional databases. |

---

### Search Engines & Inverted Indexes
Search databases use an inverted index mapping tokens/words to the documents that contain them. They enable fast full-text searching, fuzzy matching, and real-time analytical aggregations.

* **When to Use:**
  * E-commerce product search with auto-complete and filters.
  * Log aggregation and operational dashboards (e.g., ELK Stack).
  * Document classification, fuzzy matching, and geospatial search.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Blazing fast full-text search and relevance scoring. | High memory consumption and index maintenance overhead. |
| Powerful aggregation pipelines for analytics. | Not suitable as a primary transactional source of truth. |
| Supports fuzzy searching, stem matching, and synonym search. | Eventual consistency; near real-time indexing delay. |

---

### Time-Series Databases (TSDB)
Time-series databases are optimized for handling sequential data logged over time. They utilize specialized compression (e.g., Gorilla compression) and automatic partition retention policies.

* **When to Use:**
  * System monitoring metrics (CPU, Memory, Network latency).
  * Financial stock tick data and high-frequency trading.
  * IoT sensor data pipelines.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Ultra-high write ingestion rates and efficient data compression. | Poor performance for non-timestamped update operations. |
| Out-of-the-box retention policies and automatic downsampling. | Limited utility for general-purpose transactional workloads. |
| Specialized time-window aggregation functions. | High storage footprint if cardinality of tags/dimensions is too high. |

---

### Vector Databases
Vector databases store high-dimensional vector embeddings generated by machine learning models. They are designed for approximate nearest neighbor (ANN) search algorithms (e.g., HNSW, IVF).

* **When to Use:**
  * Retrieval-Augmented Generation (RAG) pipelines for Large Language Models (LLMs).
  * Semantic image, audio, and text search systems.
  * Machine-learning-based recommendation engines and personalization.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Performs sub-second similarity searches over millions of vectors. | Approximate results (trades strict accuracy for search speed). |
| Native support for distance metrics (Cosine Similarity, Euclidean, Dot Product). | High memory requirements for holding vector indexes in RAM. |
| Seamless integration with modern AI frameworks. | Specialized database engine; not built for traditional CRUD applications. |

---

## 3. System Design Interview Selection Framework

When answering database selection questions in an interview, structure your explanation using this decision workflow:

```
                          [ What is the primary access pattern? ]
                                     |
    +--------------------------------+--------------------------------+
    |                                |                                |
[Structured/ACID]              [Unstructured/Dynamic]         [Specialized Workload]
    |                                |                                |
    +---> Transactions/Joins         +---> KV Lookup: Key-Value       +---> Relationships: Graph
    |     => RDBMS (PostgreSQL)      |     => Key-Value (Redis)       |     => Graph (Neo4j)
    |                                |                                |
    +---> Massive Scale Writes       +---> Nested JSON Docs           +---> Full-Text Search
          => Wide-Column (Cassandra)       => Document (MongoDB)            => Search Engine (Elasticsearch)
                                                                      |
                                                                      +---> Time-Stamped Logs
                                                                      |     => Time-Series (TimescaleDB)
                                                                      |
                                                                      +---> AI Vector Embeddings
                                                                            => Vector DB (Pinecone)
```

### Checklist Questions to Ask in Interviews:
1. **What is the Read vs. Write ratio?** (Heavy writes $
ightarrow$ Cassandra/LSM-tree; Heavy reads $
ightarrow$ Caching/Inverted Index).
2. **Do we need strict ACID compliance?** (Yes $
ightarrow$ PostgreSQL/RDBMS; No $
ightarrow$ Document/NoSQL).
3. **What is the shape of the data?** (Hierarchical JSON $
ightarrow$ MongoDB; Graph $
ightarrow$ Neo4j; Tabular $
ightarrow$ PostgreSQL).
4. **How fast does the data grow?** (Terabytes/day $
ightarrow$ Distributed NoSQL/Wide-column).
5. **What are the latency requirements?** (Sub-millisecond $
ightarrow$ Redis/In-memory).