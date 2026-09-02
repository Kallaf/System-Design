# Comprehensive API & Network Protocol Selection Guide for System Design Interviews

Choosing the right communication protocol, message broker, and API paradigm is critical in system design. This guide compares network layer protocols, serialization formats, API architectures, message queues, event streaming platforms, and specialized domain protocols (e.g., FIX protocol, WebTransport).

---

## 1. Quick Decision Matrix

| Protocol / Paradigm | Communication Style | Data Format | Primary Use Case | Real-World Examples |
| :--- | :--- | :--- | :--- | :--- |
| **REST (HTTP/1.1 or HTTP/2)** | Synchronous Request-Response | JSON, XML | Public APIs, standard web/mobile client-server CRUD operations | Stripe API, GitHub REST API |
| **GraphQL** | Synchronous Request-Response | JSON | Complex frontends with variable data requirements, aggregated microservices | GitHub GraphQL API, Shopify |
| **gRPC** | Synchronous & Streaming RPC | Protocol Buffers (Binary) | High-performance inter-microservice (east-west) communication | Uber, Netflix internal services |
| **WebSockets** | Bi-directional Streaming | Text / Binary | Real-time bi-directional streaming, low-latency state sync | Chat apps, live trading tickers, Figma |
| **Server-Sent Events (SSE)** | Unidirectional Streaming | Text (`text/event-stream`) | Unidirectional real-time updates (server to client) | Live news feeds, LLM token streaming (ChatGPT) |
| **WebRTC** | Peer-to-Peer Streaming | Binary / Raw Media | Peer-to-peer real-time audio, video, and data channels | Zoom web, Discord voice |
| **FIX Protocol** | Persistent Session / Streaming | Key-Value (`Tag=Value`), SBE | Financial market trading (order routing, execution reports, market data) | NASDAQ, NYSE, CME, EFG Hermes |
| **Kafka / Redpanda** | Distributed Event Log (Pub/Sub) | Binary (Avro, Protobuf, JSON) | High-throughput event streaming, log aggregation, CDC pipelines | LinkedIn, Uber, Netflix |
| **RabbitMQ** | Message Queue / AMQP | Any (Binary / JSON) | Decoupled background task processing, complex message routing | Payment workflows, notification queues |
| **AWS SQS** | Distributed Queue | JSON / Text | Fully managed cloud message queue, buffering spikes | Cloud worker queues, serverless pipelines |
| **MQTT** | Lightweight Pub/Sub | Binary / JSON | Low-bandwidth IoT telemetry, constrained devices | Smart home sensors, automotive telemetry |

---

## 2. Message Queues & Event Streaming Infrastructure

### Apache Kafka / Redpanda (Distributed Event Streaming)
Kafka is an append-only distributed commit log based on topics divided into partitions. It uses a **pull-based model** where consumer groups maintain their own offsets.

* **When to Use:**
  * High-throughput event streaming (100k+ events/sec) and real-time data pipelines.
  * Change Data Capture (CDC) streaming from database WAL logs (e.g., Debezium).
  * Event-driven microservices requiring event replay capabilities.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Ultra-high write throughput (sequential disk I/O, zero-copy reads). | Messages cannot be individually deleted; retained per topic retention window. |
| Message replayability (consumers re-read logs by rewinding offsets). | Complex consumer group rebalancing and partition management. |
| Strict message ordering within a single partition. | Poor fit for complex individual message routing or immediate task scheduling. |

---

### RabbitMQ (AMQP Message Broker)
RabbitMQ is a traditional message broker built around exchanges (Direct, Fanout, Topic, Headers) and queues. It uses a **push-based model** to dispatch messages directly to active workers.

* **When to Use:**
  * Complex routing topologies (e.g., routing orders by country/priority).
  * Asynchronous background job execution (e.g., video encoding queues, email delivery).
  * Workflows requiring per-message acknowledgments (`ACK`/`NACK`), dead-letter queues (DLQ), or priority queues.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Flexible message routing using exchange bindings. | Lower throughput compared to Kafka commit logs. |
| Granular per-message control (individual ACK, dead-lettering, retries). | Messages are deleted immediately after consumption/acknowledgment (no replay). |
| Built-in support for priority queues and TTLs. | Requires careful memory management under heavy queue backlogs. |

---

### AWS SQS (Simple Queue Service)
AWS SQS is a fully managed cloud queuing service offering Standard Queues (at-least-once, unordered) and FIFO Queues (exactly-once processing, strict ordering).

* **When to Use:**
  * Cloud-native serverless architectures (e.g., triggering AWS Lambda or ECS tasks).
  * Decoupling batch workloads without managing infrastructure or brokers.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Zero server management, infinite auto-scaling. | Higher latency per request compared to self-hosted RabbitMQ/Kafka. |
| Built-in visibility timeouts and dead-letter queues. | Cost scales per API request, which can get expensive at ultra-high volume. |

---

### MQTT (Message Queuing Telemetry Transport)
MQTT is a extremely lightweight publish/subscribe protocol designed for constrained devices and high-latency or unreliable networks.

* **When to Use:**
  * IoT telemetry, embedded systems, connected vehicles, and mobile devices in low-bandwidth network environments.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Minimal packet overhead (2-byte header) and low power consumption. | Requires a central broker (single point of failure if not clustered). |
| Built-in Quality of Service (QoS 0, QoS 1, QoS 2) delivery guarantees. | Not suitable for large payload transfers or complex transactional queries. |

---

## 3. Standard API Paradigms & Protocols

### REST (Representational State Transfer)
REST is a resource-oriented architectural style built around standard HTTP methods (GET, POST, PUT, DELETE). It relies on stateless client-server interaction and uniform interfaces.

* **When to Use:**
  * External/Public-facing APIs consumed by third parties.
  * Standard CRUD operations with well-defined resources.
  * Systems leveraging aggressive HTTP caching (CDNs, browser caches).

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Stateless, widely understood, and universally supported. | Over-fetching and under-fetching of data. |
| Excellent caching mechanisms (HTTP status codes, ETag, `Cache-Control`). | Verbose text payloads (JSON) compared to binary formats. |
| Loose coupling between client and server. | Multiple round-trips (N+1 query problem) for relational data views. |

---

### GraphQL
GraphQL is a query language for APIs that lets clients request *exact* data fields. Instead of multiple endpoints, it usually exposes a single HTTP POST endpoint.

* **When to Use:**
  * Complex client frontends (mobile/web) needing composite data from multiple microservices.
  * Bandwidth-constrained environments (mobile networks) where over-fetching is costly.
  * Rapidly evolving frontend requirements without wanting constant backend endpoint changes.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Solves over-fetching and under-fetching completely. | Difficult to implement traditional HTTP-level caching. |
| Strongly typed schema (`schema.graphql`) acts as contract. | Risk of expensive/deeply nested client queries exhausting server resources. |
| Single request can fetch deeply nested, related entities. | Complex server-side execution (requires resolver optimization/DataLoader). |

---

### gRPC (Google Remote Procedure Call)
gRPC runs over **HTTP/2** and uses **Protocol Buffers (Protobuf)** for compact binary serialization. It supports unary RPCs alongside client, server, and bi-directional streaming.

* **When to Use:**
  * High-performance, low-latency microservice-to-microservice (internal/east-west) traffic.
  * Polyglot architectures requiring strict, auto-generated code/SDKs across languages.
  * Low-bandwidth environments or high-throughput internal backend backbones.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Up to 7–10x faster than REST/JSON due to compact binary serialization. | Limited direct browser support (requires gRPC-Web proxy). |
| Native HTTP/2 features (multiplexing, streaming, header compression). | Human-unreadable wire format makes manual debugging (e.g., cURL) harder. |
| Strongly typed contracts (`.proto` files) with cross-language code generation. | Rigid contract evolution (strict field numbering rules). |

---

### WebSockets
WebSockets establish a persistent, full-duplex TCP connection over a single socket after an initial HTTP handshake, allowing continuous bi-directional messaging with minimal frame overhead.

* **When to Use:**
  * Real-time collaborative applications (e.g., Figma, Google Docs).
  * Chat systems, multiplayer online gaming, and live auction platforms.
  * Systems requiring sub-50ms bi-directional communication.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Ultra-low latency full-duplex communication. | Connection state management makes horizontal scaling complex (requires sticky sessions/PubSub). |
| Eliminates HTTP header overhead after connection setup. | Harder to pass through corporate firewalls/proxies compared to standard HTTP. |
| Native browser support. | Lacks built-in reconnection logic, heartbeat management, or messaging guarantees. |

---

### Server-Sent Events (SSE)
SSE allows a server to push real-time text updates to a client over a persistent HTTP connection (`text/event-stream`). It is inherently unidirectional (Server $
ightarrow$ Client).

* **When to Use:**
  * Streaming responses from LLMs (e.g., ChatGPT output generation).
  * System alerts, live stock price updates, or continuous progress bars.
  * Simple real-time needs where client-to-server messaging can use standard HTTP POST requests.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Operates over standard HTTP; easily traverses firewalls and proxies. | Unidirectional only (Server to Client). |
| Built-in client reconnection handling and event IDs out of the box. | Native HTTP/1.1 limits browser to 6 open connections per domain (solved by HTTP/2). |
| Lightweight implementation compared to WebSockets. | Text-only payload format (binary requires Base64 encoding). |

---

### WebRTC (Web Real-Time Communication)
WebRTC enables direct peer-to-peer (P2P) audio, video, and arbitrary data streaming between browsers/clients, utilizing UDP for minimal transport latency.

* **When to Use:**
  * Real-time voice and video calls (e.g., Zoom, Google Meet).
  * Peer-to-peer file sharing applications.
  * Ultra-low latency game streaming and raw data channels.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Ultra-low latency streaming using UDP. | Complex infrastructure requirements (STUN/TURN/ICE servers for NAT traversal). |
| Direct peer-to-peer architecture reduces central server bandwidth costs. | Scaling to large multi-party calls requires complex Media Servers (SFU/MCU). |
| Built-in encryption (DTLS/SRTP). | Network quality fluctuations require sophisticated adaptive bitrate handling. |

---

## 4. Niche & Domain-Specific Protocols

### FIX Protocol (Financial Information eXchange)
FIX is an electronic communication protocol designed for international real-time exchange of securities transactions and financial market messages.

* **How it Works:** 
  * Uses tag-value ASCII format (e.g., `35=D` for New Order Single, `55=AAPL` for symbol) or **Simple Binary Encoding (SBE)** for ultra-low latency.
  * Runs over persistent TCP connections. Maintains strict session layer sequence numbering to guarantee zero message loss or duplicate trades.

* **When to Use:**
  * Order Routing Systems (ORS) connecting stock brokerages, exchanges, and institutional trading desks.
  * High-frequency trading (HFT) and algorithmic trading platforms.
  * Clearing, execution reports, and institutional market data feeds.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Industry-standard protocol across global financial exchanges. | Extremely rigid, complex session management (sequence resets, gap fills). |
| Sub-microsecond processing when using Simple Binary Encoding (SBE). | Verbose ASCII tags unless binary encoding is configured. |
| Strict deterministic execution and auditability built into protocol. | Not browser-compatible; requires custom gateway or proxy to expose to web clients. |

---

### QUIC / WebTransport
WebTransport is a modern web API using the **QUIC** protocol (UDP-based transport layer) to enable bi-directional multiplexed transport between browsers and servers.

* **When to Use:** Low-latency browser gaming, raw media ingest, avoiding TCP Head-of-Line blocking.

---

## 5. System Design Communication Decision Tree

```
                     [ What type of communication is required? ]
                                       |
    +----------------------------------+----------------------------------+
    |                                  |                                  |
[Synchronous Request-Response]   [Asynchronous / Message Queue]   [Real-Time / Streaming Push]
    |                                  |                                  |
    +---> External API: REST/GraphQL   +---> Event Replay / Log Stream    +---> Bi-directional Low Latency
    |                                  |     => Apache Kafka / Redpanda   |     => WebSockets
    +---> Internal Service: gRPC       |                                  |
                                       +---> Complex Routing / Tasks      +---> Server Push Only
                                       |     => RabbitMQ                  |     => SSE (HTTP/2)
                                       |                                  |
                                       +---> Managed Cloud Queue          +---> P2P Audio / Video
                                             => AWS SQS                         => WebRTC (UDP)
```

### Checklist Questions for Communication Design:
1. **Sync vs Async?** Does the client need an immediate response (Sync $
ightarrow$ REST/gRPC) or can work be processed later (Async $
ightarrow$ Kafka/RabbitMQ)?
2. **Replay vs Transient?** Do consumers need to replay historical events (Kafka) or discard messages once processed (RabbitMQ/SQS)?
3. **Internal vs External?** Is it consumed by public web apps (REST/GraphQL) or internal microservices (gRPC)?
