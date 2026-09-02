# Comprehensive API & Network Protocol Selection Guide for System Design Interviews

Choosing the right communication protocol and API paradigm is critical in system design. This guide compares standard and specialized network protocols, serialization formats, API architectures, and specialized niche protocols (e.g., FIX protocol for trading, gRPC-Web, WebTransport).

---

## 1. Quick Decision Matrix

| Protocol / Architecture | Underlying Transport | Data Format | Primary Use Case | Real-World Examples |
| :--- | :--- | :--- | :--- | :--- |
| **REST (HTTP/1.1 or HTTP/2)** | TCP | JSON, XML | Public APIs, standard web/mobile client-server CRUD operations | Stripe API, GitHub REST API |
| **GraphQL** | TCP (HTTP) | JSON | Complex frontends with variable data requirements, aggregated microservices | GitHub GraphQL API, Shopify |
| **gRPC** | HTTP/2 (TCP) | Protocol Buffers (Binary) | High-performance inter-microservice (east-west) communication | Uber, Netflix internal services |
| **WebSockets** | TCP | Text / Binary | Real-time bi-directional streaming, low-latency state sync | Chat apps, live trading tickers, Figma |
| **Server-Sent Events (SSE)** | HTTP / HTTP/2 | Text (`text/event-stream`) | Unidirectional real-time updates (server to client) | Live news feeds, LLM token streaming (ChatGPT) |
| **WebRTC** | UDP (primarily) | Binary / Raw Media | Peer-to-peer real-time audio, video, and data channels | Zoom web, Discord voice |
| **FIX Protocol** | TCP / SSL / Custom | Key-Value (`Tag=Value`), SBE | Financial market trading (order routing, execution reports, market data) | NASDAQ, NYSE, CME, EFG Hermes |
| **WebTransport** | QUIC (UDP) | Unreliable DATAGRAMS & Streams | Low-latency client-server messaging without head-of-line blocking | Web gaming, real-time telemetry |
| **MQTT / AMQP** | TCP | Binary / JSON | Lightweight IoT telemetry, decoupled message queuing | Smart home sensors, enterprise queues |

---

## 2. Standard API Paradigms & Protocols

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

## 3. Niche & Special Case Protocols (Interview Corner Cases)

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

* **How it Works:**
  * Combines connection setup (TLS 1.3 + Transport) into a single UDP handshake.
  * Supports both reliable streams and unreliable datagrams over a single QUIC connection.

* **When to Use:**
  * Low-latency browser games requiring out-of-order, best-effort packet delivery (unreliable datagrams).
  * High-throughput live media ingest avoiding TCP head-of-line blocking.
  * Web applications where individual packet loss shouldn't stall other concurrent streams.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Eliminates TCP Head-of-Line (HoL) blocking across multiplexed streams. | UDP traffic is sometimes throttled or blocked by strict enterprise firewalls. |
| Faster handshake latency (0-RTT connection resumption). | Newer protocol standard with evolving server framework support. |
| Supports both reliable streaming and unreliable datagrams. | Higher CPU usage on servers handling high-volume UDP encryption/decryption. |

---

### gRPC-Web
gRPC-Web allows browser applications to interact directly with gRPC backend services without requiring a full HTTP/2 stack in the browser context.

* **How it Works:**
  * Translates browser HTTP/1.1 or HTTP/2 fetch calls into gRPC protocol frames, usually passing through a proxy like **Envoy**.

* **When to Use:**
  * End-to-end type-safe frontend-to-backend communication using Protobuf definitions.
  * Web applications connecting to a gRPC microservices mesh without writing REST wrapper gateways.

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Extends Protobuf contract safety directly into web client code bases. | Requires an Envoy proxy or gateway translation layer. |
| Efficient binary transfer compared to standard JSON REST APIs. | Does not support true client-side streaming or full bi-directional streaming in browsers. |

---

### Long Polling & Short Polling (Legacy Fallbacks)
Polling relies on standard HTTP GET requests where clients repeatedly query the server for new updates.

* **Short Polling:** Client sends requests every $N$ seconds regardless of new data availability.
* **Long Polling:** Server holds the client request open until new data arrives or a timeout occurs.

* **When to Use:**
  * Fallback mechanisms when WebSockets or SSE connections fail due to proxy/firewall restrictions.
  * Asynchronous batch jobs (e.g., checking status of a video rendering pipeline).

* **Pros & Cons:**

| Pros | Cons |
| :--- | :--- |
| Simple implementation using standard HTTP libraries. | Extremely high HTTP header overhead and server load at scale. |
| Works seamlessly behind every corporate proxy and firewall. | High latency and poor resource utilization compared to WebSockets/SSE. |

---

## 4. Underlying Transport Layer Comparison

| Protocol | TCP | UDP | QUIC |
| :--- | :--- | :--- | :--- |
| **Connection Type** | Connection-oriented (3-way handshake) | Connectionless | Connection-oriented over UDP (0-RTT / 1-RTT) |
| **Reliability** | Guarantees delivery and strict ordering | Best-effort delivery; loss/reorder allowed | Per-stream reliable delivery + optional datagrams |
| **Head-of-Line Blocking** | Yes (one dropped packet stalls all data) | No | No (packet loss on one stream doesn't block others) |
| **Ideal For** | Financial transactions, web browsing, APIs | Real-time media, live voice, gaming | Modern web streaming, low-latency browser APIs |

---

## 5. System Design API Decision Framework

```
                          [ What type of communication is required? ]
                                       |
     +---------------------------------+---------------------------------+
     |                                 |                                 |
[Request - Response]              [Real-Time / Streaming]         [Specialized Industry Workloads]
     |                                 |                                 |
     +---> Internal Microservices      +---> Bi-directional Low Latency  +---> Stock Trading / Exchange Routing
     |     => gRPC (HTTP/2 + Protobuf) |     => WebSockets               |     => FIX Protocol (TCP / SBE)
     |                                 |                                 |
     +---> External / Public API       +---> Server-to-Client Push       +---> P2P Audio / Video Streaming
     |     => REST (JSON)              |     => SSE (HTTP/2)             |     => WebRTC (UDP)
     |                                 |                                 |
     +---> Complex Frontend Aggregation+---> Non-blocking Browser UDP    +---> IoT Telemetry / Low Bandwidth
           => GraphQL                        => WebTransport (QUIC)            => MQTT / AMQP
```

### Checklist Questions for API Design in Interviews:
1. **Is this an institutional or domain-specific integration?** (Stock Exchange $
ightarrow$ FIX Protocol; Web Voice $
ightarrow$ WebRTC).
2. **Is the client public or internal?** (Public $
ightarrow$ REST/GraphQL; Internal microservice $
ightarrow$ gRPC).
3. **Is packet loss acceptable for lower latency?** (Yes $
ightarrow$ WebRTC/WebTransport; No $
ightarrow$ WebSockets/TCP).
4. **Is communication unidirectional or bidirectional?** (Server push only $
ightarrow$ SSE; Bi-directional $
ightarrow$ WebSockets/gRPC streaming).
5. **Does packet loss in one stream block other streams?** (If unacceptable $
ightarrow$ QUIC / WebTransport).
