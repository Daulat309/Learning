# CLUSTER 1: CLIENT–SERVER & NETWORKING BASICS
### Topics: Client Interaction | DNS Server System

---

## TOPIC A: CLIENT INTERACTION

# 1. Concept Overview

**Definition:** Client interaction is the entire journey of how a user's device (browser, mobile app, IoT device) communicates with a backend system — from typing a URL to receiving a rendered response.

**Why it exists:** Without a structured way for clients to talk to servers, every app would invent its own ad-hoc protocol. Standardizing this (HTTP/HTTPS, REST, WebSockets) lets any client talk to any server reliably.

**Real-world analogy:** Ordering food at a restaurant. You (client) tell the waiter (request) what you want, the kitchen (server) prepares it, and the waiter brings it back (response). You don't need to know how the kitchen works internally.

**Where it is used:** Every web/mobile app — Amazon checkout, Instagram feed load, Swiggy order tracking, Zoom calls.

**Why interviewers ask it:** It's the entry point of every system design problem. If you can't explain "what happens when a user clicks a button," you can't design anything bigger.

---

# 2. Core Concepts

| Concept | Explanation |
|---|---|
| **Client** | Any device/app initiating a request (browser, mobile app, Postman, another server) |
| **Server** | A machine/program that listens for requests and sends responses |
| **Request** | A structured message asking for data/action (method, URL, headers, body) |
| **Response** | Server's reply (status code, headers, body) |
| **HTTP** | Application-layer protocol defining request/response format |
| **HTTPS** | HTTP + TLS encryption for secure communication |
| **TCP/IP** | Transport layer ensuring reliable, ordered delivery of packets |
| **Socket** | An endpoint (IP + Port) for two-way communication |
| **REST** | Architectural style using HTTP verbs (GET/POST/PUT/DELETE) on resources |
| **WebSocket** | Persistent, full-duplex connection for real-time data (chat, live scores) |
| **Polling** | Client repeatedly asks server "any update?" |
| **Long Polling** | Server holds the request open until new data is available |
| **Server-Sent Events (SSE)** | Server pushes updates to client over a single open HTTP connection |
| **Status Codes** | 2xx success, 3xx redirect, 4xx client error, 5xx server error |
| **Idempotency** | Repeating the same request has the same effect as doing it once (GET, PUT, DELETE are idempotent; POST is not) |
| **Statelessness** | Server doesn't remember previous requests; each request is self-contained (REST principle) |
| **Session/Cookie** | Mechanism to simulate state on top of stateless HTTP |
| **CORS** | Browser security rule controlling cross-origin requests |

---

# 3. Internal Working

**Simple version:** You type a URL → browser finds the server's address → sends a request → server processes it → sends back data → browser displays it.

**Technical step-by-step:**
1. User enters URL in browser.
2. Browser checks cache, then asks DNS to resolve domain → IP address.
3. Browser opens a TCP connection (3-way handshake: SYN, SYN-ACK, ACK) to that IP on port 443 (HTTPS).
4. TLS handshake happens (key exchange, certificate validation) to establish encryption.
5. Browser sends an HTTP request (method, headers, body if any).
6. Request may pass through a Load Balancer → Web/App Server.
7. Server processes (queries DB, calls other services), builds a response.
8. Response travels back over the same TCP connection.
9. Browser parses response (HTML/JSON), renders UI or returns data to the app.
10. Connection is closed or kept alive (HTTP Keep-Alive) for reuse.

---

# 4. Architecture / Flow

```
   Client (Browser/App)
          │
          │  1. DNS Lookup
          ▼
     DNS Resolver
          │
          │  2. TCP + TLS Handshake
          ▼
      Load Balancer
          │
          ▼
      Web/App Server
          │
          ▼
       Database
```

**Component explanation:**
- **Client:** Initiates request, renders response.
- **DNS Resolver:** Converts domain name to IP.
- **Load Balancer:** Distributes incoming requests across servers.
- **Web/App Server:** Executes business logic.
- **Database:** Persists/retrieves data.

---

# 5. Advantages
- **Standardization:** HTTP/REST is universally understood — any client can talk to any compliant server.
- **Statelessness:** Easy to scale horizontally since no server needs to "remember" a specific client.
- **Caching:** HTTP supports caching headers, reducing repeated work.

# 6. Disadvantages
- **Overhead:** HTTP headers and TLS handshakes add latency for every new connection.
- **Statelessness cost:** Need extra mechanisms (tokens/cookies) to maintain user sessions.
- **Polling inefficiency:** Regular polling wastes bandwidth compared to push-based mechanisms.

# 7. Trade-offs

| Approach | Best For | Downside |
|---|---|---|
| REST (HTTP) | CRUD apps, simplicity | Not real-time |
| WebSockets | Chat, gaming, live updates | Harder to scale, stateful |
| Long Polling | Near-real-time without full WebSocket complexity | Higher server load than SSE |
| SSE | One-way server push (notifications, feeds) | No client-to-server push |

---

# 8. Real-world Examples
- **Amazon:** REST APIs for browsing/checkout; WebSockets not typically needed for catalog browsing.
- **WhatsApp/Slack:** WebSockets for real-time messaging.
- **Stock trading apps:** SSE or WebSockets for live price ticks.
- **YouTube:** REST for metadata, separate streaming protocol (DASH/HLS) for video.

---

# 9. Interview Deep Dive

**Common questions:**
- What happens when you type a URL and press Enter?
- Difference between PUT and PATCH?
- Why is HTTP stateless? How do we maintain sessions then?

**Follow-ups:**
- How would you make an API idempotent?
- How does the browser know to reuse a TCP connection?

**Hidden edge cases:**
- What if DNS fails mid-request? (Fallback resolvers, cached entries)
- What if the client retries a non-idempotent POST? (Duplicate orders — needs idempotency keys)

**Frequently misunderstood:**
- People confuse "stateless" with "no session at all" — sessions exist via cookies/tokens, the *protocol* itself just doesn't track state.
- Confusing TCP handshake with TLS handshake — they're separate steps.

---

# 10. Important Terminology

| Term | Definition |
|---|---|
| Latency | Time taken for a request to get a response |
| Throughput | Number of requests handled per second |
| Handshake | Initial negotiation to establish a connection |
| Keep-Alive | Reusing a TCP connection for multiple requests |
| Payload | The actual data sent in a request/response body |

---

# 11. Time Complexity / Performance
- DNS lookup: ~20-120ms (cached: near 0ms)
- TCP handshake: 1 round trip (~RTT)
- TLS handshake: 1-2 round trips
- Throughput depends on server concurrency model (thread-per-request vs event loop)
- Scalability impact: Stateless design enables horizontal scaling with no sticky-session requirement (unless using WebSockets)

# 12. Common Mistakes
- Saying HTTP "maintains" sessions (it doesn't — cookies/tokens do).
- Forgetting TLS handshake when explaining HTTPS request flow.
- Not knowing idempotency definitions correctly (thinking POST is idempotent).

---

# 13. Revision Sheet
- Client → DNS → TCP/TLS → LB → Server → DB → Response
- REST = stateless, resource-based, uses HTTP verbs
- WebSocket = persistent, bidirectional, real-time
- Idempotent: GET, PUT, DELETE | Not idempotent: POST
- Status codes: 2xx ok, 3xx redirect, 4xx client error, 5xx server error

# 14. Memory Tricks
**"DTLSWD"** — DNS → TCP → TLS → LB → Server → DB (the request journey)
**GPD = "Get, Put, Delete are idempotent"** (mnemonic: "Good People Don't repeat mistakes")

---

# 15. Interview Questions

**Basic (10):**
1. What is the difference between client and server?
2. What is HTTP?
3. What is the difference between HTTP and HTTPS?
4. What are HTTP methods?
5. What is a status code? Give examples.
6. What is a cookie?
7. What is a session?
8. What is REST?
9. What is latency?
10. What is a socket?

**Medium (15):**
1. Explain what happens when you enter a URL in the browser.
2. Difference between PUT and PATCH.
3. What is idempotency? Which methods are idempotent?
4. Explain TCP 3-way handshake.
5. What is TLS handshake?
6. Difference between cookies and sessions.
7. What is CORS and why does it exist?
8. Explain long polling vs WebSockets.
9. What is Keep-Alive in HTTP?
10. Difference between stateful and stateless protocols.
11. What is SSE and when would you use it?
12. How do you secure an API endpoint?
13. What's the difference between authentication and authorization?
14. Explain HTTP/1.1 vs HTTP/2.
15. What is a CDN's role in client interaction?

**Advanced (10):**
1. How would you design an idempotent payment API?
2. How does HTTP/2 multiplexing improve performance?
3. Explain how WebSocket scaling works across multiple servers (sticky sessions, pub/sub backplane).
4. How would you handle a client retry storm?
5. Explain connection pooling at scale.
6. How do you handle partial failures in a multi-service request chain?
7. Explain head-of-line blocking and how HTTP/2 vs HTTP/3 address it.
8. How would you design rate limiting per client?
9. How does TLS session resumption improve performance?
10. Explain the role of API versioning in client-server contracts.

**Scenario-based (5):**
1. Your API for placing orders is called twice due to network retry — how do you prevent duplicate orders?
2. Users complain that chat messages are delayed by 5 seconds — what would you investigate?
3. Your mobile app needs offline support — how does client interaction change?
4. You need real-time stock price updates for 1M users — what would you choose: polling, SSE, or WebSocket?
5. A client reports intermittent failures — how do you debug across network layers?

**MCQs (5):**
1. Which HTTP method is NOT idempotent?
 a) GET b) PUT c) POST d) DELETE → **Answer: c) POST**
2. What does a 404 status code mean?
 a) Server error b) Not Found c) Unauthorized d) Redirect → **Answer: b) Not Found**
3. Which protocol provides full-duplex communication?
 a) HTTP b) WebSocket c) FTP d) SMTP → **Answer: b) WebSocket**
4. TLS operates at which conceptual layer?
 a) Application b) Transport/Session c) Network d) Physical → **Answer: b) Transport/Session**
5. What does CORS control?
 a) Database access b) Cross-origin requests c) Caching d) Compression → **Answer: b) Cross-origin requests**

---

# 16. Coding Connections
- **Backend Development:** Writing controllers/handlers for REST endpoints.
- **Spring Boot:** `@RestController`, `@GetMapping`, exception handlers for status codes.
- **Distributed Systems:** Request tracing across services (correlation IDs).
- **Databases:** Connection pooling tied to incoming request load.
- **Networking:** TCP/IP, sockets, TLS — foundational for every client request.
- **Operating Systems:** Socket programming, file descriptors per connection.

# 17. Related Topics
DNS, Load Balancing, API Gateway, Rate Limiting, Caching, CDN.

# 18. Cheat Sheet
```
Client → DNS → TCP+TLS → LB → Server → DB → Response
REST = stateless, resource-based
WebSocket = real-time, persistent, bidirectional
Idempotent: GET/PUT/DELETE | Non-idempotent: POST
```

# 19. Placement Rating
- Importance: ★★★★★
- Interview Frequency: ★★★★★
- Difficulty: ★★☆☆☆
- Why: Almost every system design interview opens with "what happens when a client makes a request" — it's foundational and expected knowledge.

# 20. Final Interview Summary
1. Know the full request journey: DNS → TCP/TLS → LB → Server → DB → Response.
2. REST is stateless; sessions are layered on top via cookies/tokens.
3. Know idempotency cold — it's a favorite gotcha question.
4. Know when to use REST vs WebSocket vs SSE vs polling.
5. Be ready to explain "what happens when you type a URL" in under 2 minutes.

---
---

## TOPIC B: DNS SERVER SYSTEM

# 1. Concept Overview

**Definition:** DNS (Domain Name System) is a distributed, hierarchical system that translates human-readable domain names (google.com) into machine-readable IP addresses.

**Why it exists:** Computers communicate via IP addresses, but humans can't remember numbers like 142.250.183.14. DNS acts as the internet's phonebook.

**Real-world analogy:** Like a contacts app on your phone — you save "Mom" instead of remembering her 10-digit number. DNS does this for websites.

**Where it is used:** Every single web request, email routing (MX records), CDN routing, microservice discovery internally.

**Why interviewers ask it:** Tests understanding of distributed systems, caching, and how the internet's foundational infrastructure scales globally.

---

# 2. Core Concepts

| Concept | Explanation |
|---|---|
| **Domain Name** | Human-readable address (e.g., amazon.com) |
| **IP Address** | Numeric machine address (IPv4/IPv6) |
| **DNS Resolver (Recursive Resolver)** | First stop; does the legwork of finding the IP on behalf of the client |
| **Root Server** | Top of DNS hierarchy; points to TLD servers |
| **TLD Server (Top-Level Domain)** | Handles .com, .org, .in, etc. |
| **Authoritative Name Server** | Holds the actual DNS records for a specific domain |
| **DNS Record Types** | A (IPv4), AAAA (IPv6), CNAME (alias), MX (mail), TXT (text/verification), NS (name server) |
| **TTL (Time To Live)** | How long a DNS record can be cached before re-querying |
| **DNS Caching** | Browser, OS, ISP, and resolver all cache results to reduce lookups |
| **Recursive Query** | Resolver does all the work and returns final answer to client |
| **Iterative Query** | Each server returns the next server to ask, until resolved |
| **DNS Propagation** | Time taken for DNS changes to spread across all caches globally |
| **Anycast** | Same IP advertised from multiple locations; routes to nearest one (used by root servers, CDNs) |
| **Round Robin DNS** | Returning multiple IPs in rotation for basic load distribution |
| **GeoDNS** | Returns different IPs based on user's geographic location |

---

# 3. Internal Working

**Simple version:** You type "google.com" → your computer asks a series of "phonebooks" until it finds the actual phone number (IP) → connects to that IP.

**Technical step-by-step:**
1. Browser checks its own cache for the domain.
2. If not found, checks OS-level cache (resolver cache).
3. If not found, request goes to a **Recursive DNS Resolver** (often ISP's or public like 8.8.8.8).
4. Resolver queries a **Root Server** → gets pointed to the correct **TLD Server** (e.g., .com).
5. TLD Server points to the **Authoritative Name Server** for that specific domain.
6. Authoritative server returns the actual IP address (A record).
7. Resolver caches this result (per TTL) and returns it to the browser.
8. Browser now opens a TCP connection to that IP.

---

# 4. Architecture / Flow

```
   Client
      │  "What is the IP of amazon.com?"
      ▼
 Recursive Resolver (e.g. ISP / 8.8.8.8)
      │
      ▼
  Root Server  ──────►  "Ask the .com TLD server"
      │
      ▼
  TLD Server (.com) ───►  "Ask amazon.com's authoritative server"
      │
      ▼
 Authoritative Name Server
      │
      ▼
  Returns IP Address ───► back up the chain ───► Client
```

**Component explanation:**
- **Recursive Resolver:** Acts on behalf of client, does all the querying.
- **Root Server:** Knows where TLD servers are (13 logical root server clusters globally, using Anycast).
- **TLD Server:** Knows where domain's authoritative server is.
- **Authoritative Server:** Source of truth for that domain's records.

---

# 5. Advantages
- **Scalability:** Hierarchical + cached design lets billions of devices resolve names without overloading any single server.
- **Flexibility:** Easy to change backend IPs without users needing to know (just update DNS record).
- **Load distribution:** Round Robin / GeoDNS allows basic traffic distribution at the DNS level.

# 6. Disadvantages
- **Propagation delay:** Changes can take time to reflect globally due to caching (TTL).
- **Single point of failure (if misconfigured):** If authoritative servers go down, the whole domain becomes unreachable.
- **DNS spoofing/cache poisoning:** Security vulnerability if not using DNSSEC.

# 7. Trade-offs

| Approach | Best For | Downside |
|---|---|---|
| Low TTL | Quick failover, frequent IP changes | More DNS queries, higher load |
| High TTL | Reduced load, faster repeated lookups | Slow to propagate changes |
| Round Robin DNS | Simple load distribution | No health-checking, can route to dead servers |
| Anycast | Lowest latency, used by CDNs | Complex to set up, requires BGP routing |

---

# 8. Real-world Examples
- **Google:** Uses Anycast DNS + GeoDNS to route users to the nearest data center.
- **Cloudflare:** Operates 1.1.1.1 public DNS resolver, emphasizes speed and privacy.
- **Netflix:** Uses DNS-based routing combined with CDN (Open Connect) to direct users to nearest edge server.
- **AWS Route 53:** Popular managed DNS service supporting weighted routing, latency-based routing, and health checks.

---

# 9. Interview Deep Dive

**Common questions:**
- What happens during DNS resolution?
- Difference between recursive and iterative DNS queries?
- What is TTL and why does it matter?

**Follow-ups:**
- How would you achieve zero-downtime failover using DNS?
- How does GeoDNS work?

**Hidden edge cases:**
- What happens when TTL expires mid-session? (Existing TCP connections aren't affected; only new lookups re-resolve)
- What if authoritative server is unreachable? (Resolver may serve stale cached record if configured, or fail)

**Frequently misunderstood:**
- DNS is NOT a load balancer in the traditional sense — it just hands out IPs, doesn't actively monitor traffic per request like an LB does.
- People think DNS changes are instant — they're not, due to caching/TTL.

---

# 10. Important Terminology

| Term | Definition |
|---|---|
| A Record | Maps domain to IPv4 address |
| CNAME | Maps one domain name to another (alias) |
| MX Record | Specifies mail server for a domain |
| TTL | Cache expiration time for a DNS record |
| DNSSEC | Security extension to prevent spoofing |
| Propagation | Time for DNS changes to spread globally |

---

# 11. Time Complexity / Performance
- Cold DNS lookup: 20–120ms typically (multiple hops)
- Cached lookup: ~0-1ms
- Scalability: DNS hierarchy + caching allows it to serve billions of queries/day globally
- Anycast reduces latency by routing to geographically nearest server

# 12. Common Mistakes
- Confusing DNS with a Load Balancer (DNS is coarse-grained, doesn't do real-time health checks like an LB).
- Forgetting to mention caching layers (browser, OS, ISP) when explaining lookup speed.
- Saying DNS changes are "instant" — ignoring TTL/propagation.

---

# 13. Revision Sheet
- DNS = domain name → IP address translator
- Hierarchy: Root → TLD → Authoritative
- Recursive resolver does the work for the client
- TTL controls caching duration
- Anycast/GeoDNS used for low-latency global routing
- Record types: A, AAAA, CNAME, MX, TXT, NS

# 14. Memory Tricks
**"R-T-A"** = Root → TLD → Authoritative (the resolution hierarchy)
**"CAMTeN"** = CNAME, A, MX, TXT, NS (common record types)

---

# 15. Interview Questions

**Basic (10):**
1. What is DNS?
2. Why do we need DNS instead of using IP addresses directly?
3. What is an A record?
4. What is a CNAME record?
5. What is TTL in DNS?
6. What is a recursive resolver?
7. What is an authoritative name server?
8. What is a root server?
9. What is the difference between IPv4 and IPv6?
10. What is DNS caching?

**Medium (15):**
1. Explain the full DNS resolution process step by step.
2. Difference between recursive and iterative queries.
3. What happens when TTL expires?
4. What is Round Robin DNS?
5. What is GeoDNS?
6. What is Anycast and how does it help DNS?
7. What is DNSSEC and why is it needed?
8. How does DNS help with load balancing?
9. What is propagation delay?
10. Explain MX records and their role.
11. What is a subdomain and how is it resolved?
12. How does a CDN use DNS?
13. What's the difference between public and private DNS?
14. How would you reduce DNS lookup time for a high-traffic site?
15. What is a DNS zone?

**Advanced (10):**
1. How would you design a globally distributed DNS system?
2. Explain latency-based routing in AWS Route 53.
3. How does DNS handle failover for a service outage?
4. Explain DNS cache poisoning and prevention mechanisms.
5. How do CDNs use DNS to route users to the nearest edge node?
6. What are the trade-offs of low vs high TTL in a high-availability system?
7. How would you implement blue-green deployment using DNS?
8. Explain how DNS-based service discovery works in microservices.
9. What is split-horizon DNS and when would you use it?
10. How does DNS over HTTPS (DoH) improve privacy and what trade-offs does it introduce?

**Scenario-based (5):**
1. Your website needs to migrate to a new server with zero downtime — how do you use DNS to manage the cutover?
2. Users in Asia report slow load times while US users are fine — how would DNS/CDN help?
3. You need to take a server out of rotation for maintenance — how can DNS help (and what are its limits)?
4. Your domain's authoritative server goes down — what's the impact and how do you mitigate it?
5. You're designing a multi-region active-active system — how does DNS factor into routing?

**MCQs (5):**
1. What does a CNAME record do?
 a) Maps domain to IP b) Maps domain to another domain c) Specifies mail server d) Stores text → **Answer: b**
2. Which DNS server holds the actual records for a domain?
 a) Root server b) TLD server c) Authoritative server d) Resolver → **Answer: c**
3. What does TTL control?
 a) Server load b) Cache expiration time c) Bandwidth d) Encryption → **Answer: b**
4. What technique routes users to the nearest server using the same IP?
 a) Round Robin b) Anycast c) CNAME d) MX → **Answer: b**
5. Which record type is used for email routing?
 a) A b) CNAME c) MX d) TXT → **Answer: c**

---

# 16. Coding Connections
- **Backend Development:** Configuring custom domains for APIs.
- **Spring Boot:** Service discovery (Eureka) conceptually mirrors DNS-like name resolution.
- **Distributed Systems:** Service discovery, multi-region routing.
- **Databases:** Database connection strings often use DNS-resolved hostnames (e.g., RDS endpoints).
- **Networking:** Core networking concept — UDP port 53 primarily.
- **Operating Systems:** OS-level DNS resolver cache (`/etc/resolv.conf`, `nsswitch.conf`).

# 17. Related Topics
Load Balancing, CDN, Anycast routing, Service Discovery, Client Interaction.

# 18. Cheat Sheet
```
Client → Recursive Resolver → Root → TLD → Authoritative → IP
A = IPv4 | AAAA = IPv6 | CNAME = alias | MX = mail | TXT = text
TTL = cache duration
Anycast = same IP, nearest server responds
GeoDNS = different IP based on location
```

# 19. Placement Rating
- Importance: ★★★★☆
- Interview Frequency: ★★★★☆
- Difficulty: ★★★☆☆
- Why: Frequently asked as a warm-up question or embedded inside larger "design a URL shortener / design a CDN" questions.

# 20. Final Interview Summary
1. DNS = hierarchical, distributed, cached name-to-IP translation system.
2. Resolution order: Recursive Resolver → Root → TLD → Authoritative.
3. TTL controls caching; low TTL = fast failover but more load.
4. DNS is NOT a real-time load balancer — it's coarse-grained and cache-dependent.
5. Anycast/GeoDNS are key for low-latency global routing (used heavily by CDNs).
