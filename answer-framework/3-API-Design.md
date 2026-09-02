# Comprehensive API & Network Protocol Selection Guide for System Design Interviews

Choosing the right communication protocol and API paradigm is critical in system design. This guide compares network layer protocols, serialization formats, and API architectures, detailing trade-offs, internal mechanics, and clear decision frameworks.

---

## 1. Quick Decision Matrix

| Protocol / Architecture | Underlying Transport | Data Format | Primary Use Case | Examples |
| :--- | :--- | :--- | :--- | :--- |
| **REST (HTTP/1.1 or HTTP/2)** | TCP | JSON, XML | Public APIs, standard web/mobile client-server CRUD operations | Stripe API, GitHub REST API |
| **GraphQL** | TCP (HTTP) | JSON | Complex frontends with variable data requirements, aggregated microservices | GitHub GraphQL API, Shopify |
| **gRPC** | HTTP/2 (TCP) | Protocol Buffers (Binary) | High-performance inter-microservice (east-west) communication | Uber, Netflix internal microservices |
| **WebSockets** | TCP | Text / Binary | Real-time bi-directional streaming, low-latency state sync | Chat apps, live financial tickers, collaborative editing |
| **Server-Sent Events (SSE)** | HTTP / HTTP/2 | Text (`text/event-stream`) | Unidirectional real-time updates (server to client) | Live news feeds, LLM token streaming (ChatGPT) |
| **WebRTC** | UDP (primarily) | Binary / Raw Media | Peer-to-peer real-time audio, video, and data streaming | Zoom web, Discord voice channels, WebODIN |
| **MQTT / AMQP** | TCP | Binary / JSON | Lightweight IoT telemetry, decoupled message queuing | Smart home sensors, enterprise messaging |

---

## 2. In-Depth Protocols & Architectural Paradigms

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
| Up to 7–10x faster than REST/JSON due to compact binary serialization. | Limited browser support (requires gRPC-Web proxy). |
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

## 3. Underlying Transport Layer Comparison

| Protocol | TCP | UDP |
| :--- | :--- | :--- |
| **Connection Type** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Guarantees delivery, ordering, and error checking | Best-effort delivery; packets can be lost or reordered |
| **Overhead / Speed** | Higher overhead (header size, congestion control) | Minimal overhead; maximum speed |
| **Ideal For** | APIs, web browsing, file transfers, message queues | Live audio/video, real-time gaming, DNS, telemetry |

---

## 4. API Design Decision Framework

Use this flowchart logic during system design interviews:

```
                          [ What type of communication is required? ]
                                       |
     +---------------------------------+---------------------------------+
     |                                 |                                 |
[Request - Response]              [Real-Time / Streaming]         [Peer-to-Peer / Media]
     |                                 |                                 |
     +---> Internal Microservices      +---> Bi-directional Low Latency  +---> Voice / Video / Raw P2P
     |     => gRPC (HTTP/2 + Protobuf) |     => WebSockets               |     => WebRTC (UDP)
     |                                 |                                 |
     +---> External / Public API       +---> Server-to-Client Stream     +---> IoT / Lightweight Telemetry
     |     => REST (JSON)              |     => SSE (HTTP/2)             |     => MQTT
     |                                 |
     +---> Complex Frontend Aggregation
           => GraphQL
```

### Checklist Questions to Ask in Interviews:
1. **Is the client public or internal?** (Public $
ightarrow$ REST/GraphQL; Internal $
ightarrow$ gRPC).
2. **What are the latency requirements?** (Sub-50ms bidirectional $
ightarrow$ WebSockets; Streamed responses $
ightarrow$ SSE).
3. **What is the network environment?** (Mobile/Low-bandwidth $
ightarrow$ gRPC/Protobuf or GraphQL to cut payload size).
4. **Is communication unidirectional or bidirectional?** (Server push only $
ightarrow$ SSE; Bi-directional $
ightarrow$ WebSockets/gRPC streaming).
5. **How critical is caching?** (Heavy edge caching required $
ightarrow$ REST over HTTP GET).
